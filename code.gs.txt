/**
 * OGDC PPA CamScanner Pro PDF - Backend Logic
 * Version 3.2: CamScanner Experience (Magic Color, 4-Point Perspective Warp, High-Res A4 PDF)
 */

// Konfigurasi Nama Folder & Sheet
const CONFIG = {
  FOLDER_ID: "1XzlR-X0dNjqxnoJM_ULdbFSYP9QlpoUm", // ID Folder Google Drive
  SHEET_NAME: "Data_Transaksi"                    // Nama Sheet Database
};

// 1. Fungsi Utama Render Halaman
function doGet(e) {
  return HtmlService.createHtmlOutputFromFile('Index')
    .setTitle('OGDC PPA CamScanner Pro')
    .addMetaTag('viewport', 'width=device-width, initial-scale=1, maximum-scale=1, user-scalable=0')
    .setXFrameOptionsMode(HtmlService.XFrameOptionsMode.ALLOWALL);
}

// 2. Fungsi Inisialisasi Database (Folder & Sheet)
function initDatabase() {
  const ss = SpreadsheetApp.getActiveSpreadsheet();
  let sheet = ss.getSheetByName(CONFIG.SHEET_NAME);
  
  // Buat sheet jika belum ada (Struktur Kolom: 9 Kolom)
  if (!sheet) {
    sheet = ss.insertSheet(CONFIG.SHEET_NAME);
    sheet.getRange(1, 1, 1, 9).setValues([["Timestamp", "Tanggal", "Shift", "Jenis Transaksi", "Vendor", "Berkas", "Nomer BA", "Nama File PDF", "Link Download"]]);
    sheet.getRange(1, 1, 1, 9).setFontWeight("bold").setBackground("#f3f4f6");
    sheet.setFrozenRows(1);
  }

  // Ambil folder secara spesifik berdasarkan ID
  const folder = DriveApp.getFolderById(CONFIG.FOLDER_ID);
  
  return { sheet: sheet, folder: folder };
}

// 3. Proses Simpan Transaksi & Generate PDF
function saveTransaction(formData, imagesBase64, uploadedPdfBase64 = null) {
  try {
    const db = initDatabase();
    
    // Setup Waktu (Mandatory Timezone WITA / Asia/Makassar)
    const now = new Date();
    const timestamp = Utilities.formatDate(now, 'Asia/Makassar', 'dd/MM/yyyy HH:mm:ss');
    // Sanitasi Nomer BA untuk nama file agar tidak ada karakter ilegal (seperti / atau \)
    const safeNomerBa = (formData.nomerBa || 'DOC').replace(/[^a-zA-Z0-9-]/g, '_');
    const safeVendor = (formData.vendor || 'VENDOR').replace(/[^a-zA-Z0-9-]/g, '_');
    const safeBerkas = (formData.berkas || 'BERKAS').replace(/\s+/g, '');
    const fileName = `SCAN_${safeVendor}_${safeBerkas}_${safeNomerBa}_${Utilities.formatDate(now, 'Asia/Makassar', 'yyyyMMdd_HHmmss')}.pdf`;

    let pdfFile;
    let pdfUrl;

    if (uploadedPdfBase64) {
      // HANDLE DIRECT PDF UPLOAD (Dari jsPDF atau upload PDF langsung)
      const base64Data = uploadedPdfBase64.split(',')[1] || uploadedPdfBase64;
      const decodedData = Utilities.base64Decode(base64Data);
      const blob = Utilities.newBlob(decodedData, MimeType.PDF, fileName);
      
      pdfFile = db.folder.createFile(blob);
      // Atur izin sharing agar link bisa dibuka langsung
      try {
        pdfFile.setSharing(DriveApp.Access.ANYONE_WITH_LINK, DriveApp.Permission.VIEW);
      } catch (e) {
        console.warn("Set sharing warning: " + e.toString());
      }
      pdfUrl = pdfFile.getUrl();

    } else if (imagesBase64 && imagesBase64.length > 0) {
      // FALLBACK: buat PDF scan gambar melalui HTML
      let htmlContent = `
        <!DOCTYPE html>
        <html>
        <head>
          <meta charset="UTF-8">
          <style>
            @page { size: A4; margin: 0; }
            html, body { margin: 0; padding: 0; background: #fff; }
            .scan-page {
              box-sizing: border-box;
              width: 210mm;
              height: 297mm;
              page-break-after: always;
              display: flex;
              align-items: center;
              justify-content: center;
              overflow: hidden;
              background: #fff;
            }
            .scan-page:last-child { page-break-after: auto; }
            .scan-page img {
              display: block;
              max-width: 207mm;
              max-height: 294mm;
              width: auto;
              height: auto;
              object-fit: contain;
            }
          </style>
        </head>
        <body>
      `;

      imagesBase64.forEach((base64, index) => {
        htmlContent += `
          <div class="scan-page">
            <img src="${base64}" alt="Halaman scan ${index + 1}">
          </div>
        `;
      });

      htmlContent += `</body></html>`;
      const blob = Utilities.newBlob(htmlContent, MimeType.HTML).getAs(MimeType.PDF);
      blob.setName(fileName);
      pdfFile = db.folder.createFile(blob);
      try {
        pdfFile.setSharing(DriveApp.Access.ANYONE_WITH_LINK, DriveApp.Permission.VIEW);
      } catch (e) {
        console.warn("Set sharing warning: " + e.toString());
      }
      pdfUrl = pdfFile.getUrl();
    } else {
      throw new Error("Tidak ada data file PDF atau gambar yang dikirim.");
    }

    // BATCH PROCESSING: SAVE TO SPREADSHEET
    const lastRow = db.sheet.getLastRow();
    const dataToSave = [[
      timestamp, formData.tanggal, formData.shift, formData.jenis, 
      formData.vendor, formData.berkas, formData.nomerBa, 
      fileName, pdfUrl
    ]];
    
    db.sheet.getRange(lastRow + 1, 1, 1, 9).setValues(dataToSave);
    SpreadsheetApp.flush(); // MANDATORY: Real-Time Sync

    return { success: true, message: "Dokumen berhasil disimpan ke Drive & Sheet!", pdfUrl: pdfUrl, fileName: fileName };
  } catch (error) {
    return { success: false, message: "Error Code.gs: " + error.toString() };
  }
}

// 4. Fungsi Mengambil Riwayat (Untuk Render Frontend)
function getHistoryData() {
  try {
    const ss = SpreadsheetApp.getActiveSpreadsheet();
    const sheet = ss.getSheetByName(CONFIG.SHEET_NAME);
    
    if (!sheet) return []; 
    
    const lastRow = sheet.getLastRow();
    if (lastRow <= 1) return []; // Kosong
    
    const data = sheet.getRange(2, 1, lastRow - 1, 9).getValues();
    
    return data.map((row, index) => {
      let ts = row[0];
      if (ts instanceof Date) ts = Utilities.formatDate(ts, 'Asia/Makassar', 'dd/MM/yyyy HH:mm:ss');

      let tgl = row[1];
      if (tgl instanceof Date) {
        tgl = Utilities.formatDate(tgl, 'Asia/Makassar', 'dd/MM/yyyy');
      } else if (typeof tgl === 'string' && tgl.includes('-')) {
        const parts = tgl.split('-');
        if (parts.length === 3) tgl = `${parts[2]}/${parts[1]}/${parts[0]}`;
      }

      return {
        rowId: index + 2,
        timestamp: ts ? ts.toString() : '',
        tanggal: tgl ? tgl.toString() : '',
        shift: row[2] ? row[2].toString() : '',
        jenis: row[3] ? row[3].toString() : '',
        vendor: row[4] ? row[4].toString() : '',
        berkas: row[5] ? row[5].toString() : '',
        nomerBa: row[6] ? row[6].toString() : '', 
        fileName: row[7] ? row[7].toString() : '',
        pdfUrl: row[8] ? row[8].toString() : ''
      };
    }).reverse();

  } catch (error) {
    throw new Error("Gagal mengambil riwayat: " + error.toString());
  }
}

// 5. Fungsi Edit Data Transaksi
function updateTransaction(rowId, formData) {
  try {
    const ss = SpreadsheetApp.getActiveSpreadsheet();
    const sheet = ss.getSheetByName(CONFIG.SHEET_NAME);
    
    const dataToUpdate = [[
      formData.tanggal, formData.shift, formData.jenis, 
      formData.vendor, formData.berkas, formData.nomerBa
    ]];
    
    sheet.getRange(rowId, 2, 1, 6).setValues(dataToUpdate);
    SpreadsheetApp.flush(); // MANDATORY
    
    return { success: true, message: "Data riwayat berhasil diperbarui!" };
  } catch (error) {
    return { success: false, message: "Gagal update riwayat: " + error.toString() };
  }
}

// 6. Fungsi Hapus Transaksi & PDF
function deleteTransaction(rowId, pdfUrl) {
  try {
    const ss = SpreadsheetApp.getActiveSpreadsheet();
    const sheet = ss.getSheetByName(CONFIG.SHEET_NAME);
    
    // 1. Ekstrak ID File dari URL PDF dan Hapus dari Google Drive
    if (pdfUrl) {
      const match = pdfUrl.match(/[-\w]{25,}/);
      if (match && match[0]) {
        const fileId = match[0];
        try {
          const file = DriveApp.getFileById(fileId);
          file.setTrashed(true); // Masukkan ke tong sampah Drive (bisa dipulihkan)
        } catch(e) {
          console.warn("File PDF tidak ditemukan atau sudah terhapus di Drive.");
        }
      }
    }

    // 2. Hapus Baris di Database (Sheet)
    sheet.deleteRow(rowId);
    SpreadsheetApp.flush();
    
    return { success: true, message: "Data dan file PDF berhasil dihapus!" };
  } catch (error) {
    return { success: false, message: "Gagal menghapus data: " + error.toString() };
  }
}