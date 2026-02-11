# 22 - Entrega y Cierre (Offboarding)

> **Objetivo:** Terminar la relación profesionalmente, cobrar lo pendiente y evitar llamadas de soporte eterno.

---

## Checklist de Cierre

### 1. Infraestructura y Accesos
*   [ ] **Transferencia de DNS:** Mover dominio a la cuenta del cliente (Cloudflare/Namecheap).
*   [ ] **Facturación:** Asegurar que la tarjeta de crédito en AWS/Vercel/Neon sea la del cliente.
*   [ ] **Rotación de Credenciales:**
    *   Cambiar claves de BD.
    *   Regenerar API Keys de terceros (Stripe, SendGrid).
    *   Eliminar tu usuario "Admin" o cambiarle la contraseña.
    *   Eliminarte del repositorio GitHub (o transferirlo si fue venta de código).

### 2. Documentación (El "Entregable")
Debes entregar un ZIP o carpeta Drive con:
1.  **Manual de Usuario:** PDF simple con screenshots (ver `docs/19-DOCUMENTACION-PROYECTO.md`).
2.  **Manual Técnico:** `README.md` + Credenciales (en archivo seguro como 1Password o PDF encriptado).
3.  **Código Fuente:** Link al repo o ZIP (según contrato).
4.  **Licencia:** Archivo `LICENSE` claro.

### 3. Garantía (Soporte Post-Venta)
Define explícitamente qué cubre la garantía:

> *"El desarrollador corregirá **Bugs Bloqueantes** (errores 500, flujos rotos) reportados durante los primeros **30 días** tras la entrega. Cambios estéticos o nuevas funcionalidades se cotizan aparte."*

---

## El Correo Final

```text
Asunto: [Proyecto] Entrega Final y Cierre

Hola [Cliente],

Adjunto los accesos finales y documentación del proyecto.
A partir de hoy, el sistema es 100% propiedad suya.

Resumen:
1. El dominio ya apunta a su cuenta.
2. La facturación está a su nombre.
3. Tiene 30 días de garantía para reportar errores críticos.

Ha sido un gusto trabajar juntos. Para futuros desarrollos, mi tarifa es de [X] UF/hora.

Saludos,
[Tu Nombre]
```
