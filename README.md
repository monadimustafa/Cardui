import { Injectable } from '@angular/core';
import { PDFDocument, rgb, StandardFonts } from 'pdf-lib';

@Injectable({
  providedIn: 'root'
})
export class PdfGeneratorService {

  async generatePdf(): Promise<Uint8Array> {
    const pdfDoc = await PDFDocument.create();
    const page = pdfDoc.addPage([600, 800]);
    const { width, height } = page.getSize();

    const font = await pdfDoc.embedFont(StandardFonts.Helvetica);
    const fontSize = 12;

    // Logo (Remplacez par votre propre image en Base64)
    const logoUrl = 'data:image/png;base64,IMAGE_BASE64_HERE';
    const logoBytes = await fetch(logoUrl).then(res => res.arrayBuffer());
    const logoImage = await pdfDoc.embedPng(logoBytes);
    page.drawImage(logoImage, {
      x: 50,
      y: height - 100,
      width: 80,
      height: 80,
    });

    // Objet et Date
    page.drawText('Objet: Demande de clôture de compte', {
      x: 150,
      y: height - 60,
      size: 14,
      font,
      color: rgb(0, 0, 0),
    });

    page.drawText(`Date de création: ${new Date().toLocaleDateString()}`, {
      x: 150,
      y: height - 80,
      size: fontSize,
      font,
      color: rgb(0, 0, 0),
    });

    // Texte principal
    page.drawText('Cher(e) Responsable,\nJe vous prie de bien vouloir procéder à la clôture de mon compte bancaire selon les détails suivants :', {
      x: 50,
      y: height - 140,
      size: fontSize,
      font,
      color: rgb(0, 0, 0),
    });

    // Tableau RIB et motif
    const tableY = height - 200;
    page.drawText('RIB', { x: 50, y: tableY, size: fontSize, font, color: rgb(0, 0, 0), });
    page.drawText('Motif de clôture', { x: 250, y: tableY, size: fontSize, font, color: rgb(0, 0, 0), });
    
    page.drawText('12345678901234567890', { x: 50, y: tableY - 20, size: fontSize, font });
    page.drawText('Changement de banque', { x: 250, y: tableY - 20, size: fontSize, font });

    page.drawText('09876543210987654321', { x: 50, y: tableY - 40, size: fontSize, font });
    page.drawText('Fermeture définitive', { x: 250, y: tableY - 40, size: fontSize, font });

    // Signature
    page.drawText('Signature :', { x: 50, y: tableY - 100, size: fontSize, font, color: rgb(0, 0, 0), });

    // Générer le PDF
    const pdfBytes = await pdfDoc.save();
    return pdfBytes;
  }
}
