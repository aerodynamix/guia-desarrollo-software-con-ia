# 19 - Pasarelas de Pago

> **Regla:** Nunca toques los números de la tarjeta de crédito. Usa Stripe Elementos o Checkout.

---

## 1. Patrones Técnicos

### Webhooks (Asíncronos)
El pago **NO** termina cuando el usuario hace clic. Termina cuando el banco avisa.
1. Usuaro paga en Frontend.
2. Pasarela procesa (puede tardar segundos o días).
3. Pasarela envía **Webhook** (POST) a tu Backend.
4. Tu Backend valida la firma y actualiza la BD (`status: 'paid'`).

**Seguridad de Webhooks:**
*   Valida la **Firma Criptográfica** (Stripe-Signature). No confíes en el body solo.
*   Retorna `200 OK` rápido. Si fallas, la pasarela reintentará (y podrías procesar doble).

### Idempotencia (Evitar cobros dobles)
Si el webhook llega 2 veces por error de red:
*   Usa el `id_transaccion` de la pasarela como llave única.
*   Antes de procesar, verifica: `if (pago.status === 'paid') return;`

### Reconciliación
Una vez al mes, corre un script que compare:
`Total en BD` vs `Total en Stripe`.
Las diferencias ocurren (refunds manuales, chargebacks).

---

## 2. Proveedores (Latam vs Global)

| Proveedor | Comisión Aprox | Ventaja | Desventaja |
|-----------|----------------|---------|------------|
| **Stripe** | 2.9% + $0.30 | Deber Developer Experience (DX). | Requiere empresa en USA/EU/Mex/Bra (difícil en Chile/Arg). |
| **MercadoPago** | ~3.5% | Líder en Latam. Acepta todo. | DX regular. Documentación a veces desactualizada. |
| **Fintoc** | ~1.5% | Pagos directos banco a banco (Chile). | Solo Chile (por ahora). Menos fricción que tarjeta. |
| **PayPal** | 5.4% | Global. | Comisiones altas. Retiro de dinero lento. |

---

## 3. Webhook Controller (Ejemplo Conceptual)

```typescript
@Post('webhook/stripe')
async handleStripeWebhook(@Headers('stripe-signature') sig, @Body() rawBody) {
  // 1. Validar firma (Critical!)
  const event = stripe.webhooks.constructEvent(rawBody, sig, endpointSecret);

  // 2. Switch por tipo de evento
  switch (event.type) {
    case 'checkout.session.completed':
      await financeService.confirmarPago(event.data.object.userId);
      break;
    case 'invoice.payment_failed':
      await financeService.pausarSuscripcion(event.data.object.subscriptionId);
      break;
  }

  return { received: true };
}
```
