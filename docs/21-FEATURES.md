# 21 - Funcionalidades Comunes

> **Objetivo:** No reinventar la rueda. Usa estas librerías estándar para tareas comunes.

---

## 1. Envío de Correos (Transaccionales)

**Stack Recomendado (NestJS):**
*   **Servicio:** Resend (Mejor DX), SendGrid o AWS SES.
*   **Templating:** **React Email** (Genera HTML compatible con Outlook/Gmail).
*   **Libreria:** `nest-modules/mailer` o integración directa.

**Ejemplo (Resend + React Email):**
```typescript
import { Resend } from 'resend';
import { WelcomeEmail } from './emails/welcome'; // Componente React renderizado a string

const resend = new Resend(process.env.RESEND_API_KEY);

await resend.emails.send({
  from: 'Soporte <hola@miempresa.com>',
  to: 'cliente@gmail.com',
  subject: 'Bienvenido!',
  html: render(WelcomeEmail({ nombre: 'Juan' })),
});
```

---

## 2. Generación de Documentos (PDF / Excel)

### PDF (Facturas, Reportes)
*   **Simple:** `jspdf` (Generar PDF en cliente desde Angular).
*   **Complejo/Pixel-Perfect:** `puppeteer` o `playwright` (Renderiza HTML/CSS a PDF en servidor NestJS).
    *   *Tip:* Diseña la factura en HTML + Tailwind y usa Puppeteer para imprimirla.

### Excel / CSV
*   **Librería:** `xlsx` (SheetJS) o `exceljs`.
*   **Uso (Angular):**
    ```typescript
    import * as XLSX from 'xlsx';
    
    exportToExcel(data: any[]) {
      const ws = XLSX.utils.json_to_sheet(data);
      const wb = XLSX.utils.book_new();
      XLSX.utils.book_append_sheet(wb, ws, "Ventas");
      XLSX.writeFile(wb, "reporte_ventas.xlsx");
    }
    ```

---

## 3. Visualización de Datos (Gráficos)

*   **Librerías Angular:**
    *   `ngx-charts`: Data visualization declarativa para Angular (SVG).
    *   `ng2-charts` (Chart.js wrapper): Canvas-based, simple y efectivo.
    *   `apexcharts-ng`: Interactivos y modernos.

**Regla:** Asegura el contraste de colores para daltónicos.

---

## 4. Códigos QR y Barras

### Código QR (Entradas, Pagos)
*   **Librería:** `angularx-qrcode`.
*   **Componente:**
    ```html
    <qrcode [qrdata]="'Mis Datos'" [width]="256" [errorCorrectionLevel]="'M'"></qrcode>
    ```

### Código de Barras
*   **Librería:** `jsbarcode`.
*   **Formatos:** CODE128 (General), EAN (Productos retail).

---

## 5. Manejo de Fechas

**Librería Estándar:** `date-fns`.
Evitar `moment.js` (deprecated y pesado).

```typescript
import { format, subDays } from 'date-fns';
import { es } from 'date-fns/locale';

const fecha = format(new Date(), 'PPPP', { locale: es });
// "viernes, 29 de abril de 2026"
```
