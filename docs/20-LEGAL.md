# 20 - ASPECTOS LEGALES, CONTRATOS Y LICENCIAS (REFORZADO)

> "Los contratos claros protegen relaciones largas."  
> Documenta TODO antes de comenzar. Un buen contrato previene conflictos futuros.

---

## SECCION 1: CONTRATOS FREELANCE (PROPUESTA COMERCIAL)

### 1.1 Estructura Obligatoria de Propuesta

Antes de escribir una sola línea de código, el supervisor DEBE enviar al cliente un documento que incluya:

#### A) ALCANCE DEL PROYECTO (SCOPE)

**QUÉ INCLUYE:**
```markdown
El proyecto incluye:
- [X] Diseño de interfaz de usuario (UI/UX) responsive
- [X] Desarrollo de frontend (tecnología especificada)
- [X] Desarrollo de backend con API RESTful
- [X] Base de datos PostgreSQL con modelo de datos normalizado
- [X] Sistema de autenticación (JWT + Refresh Tokens)
- [X] Panel de administración
- [X] Documentación técnica para desarrolladores
- [X] Manual de usuario en formato PDF
- [X] Deploy en servidor del cliente (1 ambiente: producción)
```

**QUÉ NO INCLUYE (CRÍTICO):**
```markdown
El proyecto NO incluye:
- [ ] Soporte post-entrega fuera del período de garantía
- [ ] Modificaciones al alcance después de la firma
- [ ] Integración con sistemas externos no especificados
- [ ] Redacción de contenido (textos, imágenes)
- [ ] Hosting o dominio (responsabilidad del cliente)
- [ ] Capacitación presencial
- [ ] Migraciones de datos de sistemas legacy
```

#### B) CRONOGRAMA Y HITOS

| Hito | Entregable | Fecha Estimada | Pago |
|------|------------|----------------|------|
| H1 | Diseño UI/UX aprobado + Arquitectura técnica | Semana 2 | 30% |
| H2 | Frontend funcional + Backend API (80%) | Semana 4 | 30% |
| H3 | Testing completo + Deploy producción | Semana 6 | 30% |
| H4 | Entrega final + Documentación | Semana 7 | 10% |

**NOTA:** Las fechas son ESTIMADAS. Retrasos del cliente (feedback tardío, cambios de alcance) extienden el cronograma proporcionalmente.

#### C) INVERSIÓN Y FORMA DE PAGO

**Desglose de Costos:**
```
Desarrollo Frontend:    $X,XXX,XXX CLP
Desarrollo Backend:     $X,XXX,XXX CLP
Infraestructura (1 mes):   $XX,XXX CLP
Documentación:            $XXX,XXX CLP
──────────────────────────────────
TOTAL:                  $X,XXX,XXX CLP
```

**Forma de Pago:**
1. **30% Anticipo:** Antes de comenzar (reserva tu espacio en agenda)
2. **30% Hito 2:** Al completar frontend + backend funcional
3. **30% Hito 3:** Al deploy en producción y testing completo
4. **10% Final:** Al entregar documentación y código fuente

**Métodos de Pago Aceptados:**
- Transferencia bancaria (Chile: cuenta corriente/vista)
- PayPal (comisión +5% asumida por cliente)
- Stripe (comisión +3.5% asumida por cliente)

---

## SECCION 2: PROPIEDAD INTELECTUAL (IP)

### 2.1 Cláusula de Transferencia de IP

**MODELO ESTÁNDAR (Recomendado para Freelance):**

```markdown
CLÁUSULA 4: PROPIEDAD INTELECTUAL

4.1 TRABAJO CUSTOM:
El código fuente desarrollado específicamente para este proyecto se 
transfiere al CLIENTE una vez realizado el pago COMPLETO del 100% del monto acordado.

4.2 CÓDIGO REUTILIZABLE:
El PROVEEDOR (freelancer) retiene derechos sobre:
- Librerías propias desarrolladas previamente
- Componentes genéricos reutilizables (formularios, autenticación base)
- Frameworks y herramientas de código abierto utilizadas

4.3 LICENCIAS DE TERCEROS:
Todo software de terceros (Node.js, PostgreSQL, Vue.js, etc.) se rige 
por sus propias licencias open-source (MIT, Apache 2.0, GPL).

4.4 USO EN PORTFOLIO:
El PROVEEDOR puede:
- Mostrar capturas de pantalla del proyecto en su portafolio
- Mencionar el nombre del cliente (salvo NDA explícito)
- NO puede revelar código fuente ni datos sensibles del cliente
```

### 2.2 Cláusula de Código Generado por IA

**CRÍTICO: DISCLAMER OBLIGATORIO EN TODO CONTRATO 2024+**

```markdown
CLÁUSULA 5: USO DE HERRAMIENTAS DE INTELIGENCIA ARTIFICIAL

5.1 TRANSPARENCIA:
El PROVEEDOR utiliza herramientas de asistencia basadas en IA (GitHub Copilot, 
Claude, ChatGPT, etc.) para acelerar el desarrollo. Esto NO afecta la calidad 
ni la propiedad del código entregado.

5.2 RESPONSABILIDAD:
El PROVEEDOR es RESPONSABLE de:
- Revisar y validar todo código generado por IA
- Garantizar que el código cumple con requisitos funcionales y de seguridad
- Asegurar que no se infringen licencias de terceros

5.3 GARANTÍA DE ORIGINALIDAD:
El PROVEEDOR garantiza que el código entregado:
- No contiene plagio de código propietario de terceros
- No viola patentes ni derechos de autor conocidos
- Ha sido revisado manualmente para evitar vulnerabilidades de seguridad

5.4 AUDITORÍA:
El CLIENTE puede solicitar auditoría de terceros para verificar la calidad 
del código (costo asumido por quien solicita).
```

---

## SECCION 3: CONFIDENCIALIDAD (NDA)

### 3.1 Acuerdo de No Divulgación Mutuo

**MODELO DE NDA SIMPLIFICADO:**

```markdown
ACUERDO DE CONFIDENCIALIDAD

Entre:
- PROVEEDOR: [Tu nombre/empresa]
- CLIENTE: [Nombre del cliente]

INFORMACIÓN CONFIDENCIAL incluye:
- Modelos de negocio y estrategias comerciales
- Bases de datos con información de usuarios
- Código fuente propietario
- Documentación técnica interna
- Credenciales de acceso (API Keys, passwords)

OBLIGACIONES:
1. Ambas partes se comprometen a NO divulgar información confidencial a terceros
2. La información solo se usará para el propósito de este proyecto
3. Al finalizar el proyecto, el PROVEEDOR eliminará credenciales de acceso
4. Vigencia: Durante el proyecto + 2 años post-finalización

EXCEPCIONES (información NO confidencial):
- Información de dominio público
- Información obtenida legalmente de terceros
- Información que el receptor ya poseía antes del acuerdo
```

---

## SECCION 4: GARANTÍA Y SOPORTE POST-ENTREGA

### 4.1 Período de Garantía

**MODELO ESTÁNDAR:**

```markdown
CLÁUSULA 6: GARANTÍA Y SOPORTE

6.1 GARANTÍA DE BUGS CRÍTICOS:
El PROVEEDOR ofrece 60 días de garantía POST-ENTREGA para:
- Bugs críticos que impidan el uso del sistema (P0/P1)
- Errores de seguridad (vulnerabilidades)
- Fallos de funcionalidades principales documentadas

6.2 EXCLUSIONES DE GARANTÍA:
NO están cubiertos por la garantía:
- Cambios de alcance o nuevas funcionalidades
- Problemas causados por modificaciones del cliente
- Errores de infraestructura (servidor caído, BD corrupta)
- Solicitudes de cambio de diseño o UI

6.3 TIEMPOS DE RESPUESTA:
- P0 (Crítico - Sistema caído): 24 horas
- P1 (Alto - Funcionalidad principal afectada): 48 horas
- P2 (Medio - Bug menor): 7 días
- P3 (Bajo - Mejora cosmética): Fuera de garantía

6.4 SOPORTE EXTENDIDO (OPCIONAL):
Después del período de garantía, el cliente puede contratar:
- Plan Mensual: $XXX,XXX CLP/mes (hasta 10 horas de soporte)
- Por Incidente: $XX,XXX CLP/hora
```

---

## SECCION 5: CLÁUSULAS DE PROTECCIÓN PARA EL FREELANCER

### 5.1 Protección Contra Scope Creep

```markdown
CLÁUSULA 7: CAMBIOS AL ALCANCE

7.1 SOLICITUDES DE CAMBIO:
Cualquier modificación al alcance original requiere:
- Solicitud por escrito (email)
- Evaluación de impacto en tiempo y costo
- Firma de addendum al contrato
- Pago adicional si incrementa esfuerzo

7.2 CAMBIOS MENORES:
Se consideran cambios menores (sin costo extra):
- Ajustes de texto o colores
- Corrección de bugs introducidos por el PROVEEDOR
- Cambios que tomen menos de 2 horas de trabajo

7.3 CAMBIOS MAYORES:
Se consideran cambios mayores (requieren pago adicional):
- Nueva funcionalidad no especificada
- Rediseño de módulos existentes
- Integración con servicios externos no contemplados
- Migración de datos de sistemas legacy
```

### 5.2 Protección Contra Pagos Tardíos

```markdown
CLÁUSULA 8: PENALIDADES POR PAGO TARDÍO

8.1 INTERÉS MORATORIO:
Pagos atrasados generan interés del 2% mensual sobre el monto pendiente.

8.2 SUSPENSIÓN DE TRABAJO:
Si el pago de un hito se retrasa más de 7 días, el PROVEEDOR puede:
- Suspender el trabajo hasta recibir el pago
- Extender el cronograma proporcionalmente

8.3 RESCISIÓN POR INCUMPLIMIENTO:
Si el cliente no paga después de 30 días de vencido el plazo:
- El PROVEEDOR puede rescindir el contrato
- El código desarrollado hasta ese momento NO se entrega
- El PROVEEDOR retiene los pagos recibidos como compensación
```

---

## SECCION 6: LICENCIAS DE SOFTWARE

### 6.1 Opciones de Licencia para Código Entregado

Cuando entregas el código al cliente, puedes licenciarlo bajo:

#### A) LICENCIA PROPIETARIA (Uso Exclusivo del Cliente)

```markdown
LICENCIA DE SOFTWARE PROPIETARIO

Copyright (c) 2026 [Tu Nombre]

Se concede al CLIENTE el derecho exclusivo de:
- Usar el software en su negocio
- Modificar el código fuente
- Crear trabajos derivados

El CLIENTE NO puede:
- Revender el software a terceros como producto
- Publicar el código fuente bajo licencia open-source
- Sublicenciar a terceros sin autorización escrita

Esta licencia es PERPETUA pero requiere pago completo del proyecto.
```

#### B) LICENCIA MIT (Si Quieres Reutilizar el Código)

```markdown
MIT License

Copyright (c) 2026 [Tu Nombre]

Permission is hereby granted, free of charge, to any person obtaining a copy
of this software and associated documentation files (the "Software"), to deal
in the Software without restriction, including without limitation the rights
to use, copy, modify, merge, publish, distribute, sublicense, and/or sell
copies of the Software...

[Texto completo de MIT License]
```

**CUÁNDO USAR CADA LICENCIA:**
- **Propietaria:** Cliente paga por exclusividad, tú no puedes reutilizar el código
- **MIT:** Tú puedes reutilizar componentes en futuros proyectos, cliente tiene libertad total

---

## SECCION 7: RESCISIÓN Y TERMINACIÓN

### 7.1 Causas de Rescisión Justificada

```markdown
CLÁUSULA 9: TERMINACIÓN DEL CONTRATO

9.1 RESCISIÓN POR EL PROVEEDOR:
El PROVEEDOR puede terminar el contrato si:
- Cliente no paga 2 hitos consecutivos
- Cliente no provee información/accesos necesarios después de 3 solicitudes
- Cliente solicita acciones ilegales (piratería, spam, etc.)

9.2 RESCISIÓN POR EL CLIENTE:
El CLIENTE puede terminar el contrato si:
- PROVEEDOR incumple plazos por más de 4 semanas sin justificación
- El código entregado no cumple con funcionalidades acordadas

9.3 RESCISIÓN AMISTOSA:
Ambas partes pueden acordar terminación anticipada con:
- Pago proporcional al trabajo completado
- Entrega del código desarrollado hasta ese momento
- Cancelación de obligaciones futuras
```

---

## SECCION 8: JURISDICCIÓN Y LEY APLICABLE

### 8.1 Para Clientes en Chile

```markdown
CLÁUSULA 10: JURISDICCIÓN

Este contrato se rige por las leyes de la República de Chile.

Cualquier disputa se resolverá mediante:
1. PRIMERO: Mediación voluntaria (30 días)
2. SEGUNDO: Arbitraje vinculante
3. TERCERO: Tribunales de [Ciudad, Chile]

Ambas partes renuncian a litigios internacionales.
```

### 8.2 Para Clientes Internacionales

```markdown
CLÁUSULA 10: JURISDICCIÓN INTERNACIONAL

Dado que este es un contrato de servicios remotos:
- Ley aplicable: [País del freelancer]
- Arbitraje: Plataforma de resolución de disputas online (ej: PayPal, Upwork)
- Idioma del contrato: Español (con traducción al inglés si es necesario)
```

---

## SECCION 9: INSTRUCCIONES PARA EL AGENTE IA

### 9.1 Generación Automática de Contratos

Cuando el supervisor solicite generar un contrato, DEBES:

1. **Preguntar Datos Clave:**
   ```
   - Nombre del cliente:
   - RUT/RFC/Tax ID del cliente:
   - Tipo de proyecto:
   - Monto total:
   - Duración estimada:
   - ¿Requiere NDA?:
   - ¿Código propietario o MIT?:
   ```

2. **Generar Documento en Markdown:**
   ```markdown
   # CONTRATO DE DESARROLLO DE SOFTWARE
   
   Entre:
   PROVEEDOR: [Datos del freelancer]
   CLIENTE: [Datos del cliente]
   
   FECHA: [Fecha actual]
   
   [Incluir todas las cláusulas relevantes]
   ```

3. **Exportar a DOCX/PDF:**
   - Usa la skill de `docx` para generar un .docx profesional
   - Incluye portada, índice, y firmas digitales

### 9.2 Checklist Pre-Firma

Antes de que el supervisor envíe el contrato al cliente:

- [ ] Monto total y forma de pago especificados
- [ ] Alcance detallado (qué incluye / qué NO incluye)
- [ ] Cronograma con hitos y fechas
- [ ] Cláusula de IP y código generado por IA
- [ ] NDA si el cliente maneja datos sensibles
- [ ] Garantía de bugs críticos (60 días estándar)
- [ ] Penalidades por pago tardío
- [ ] Causas de rescisión
- [ ] Jurisdicción y ley aplicable

---

## SECCION 10: FACTURACIÓN Y IMPUESTOS (CHILE)

### 10.1 Emisión de Boletas de Honorarios

**Para Freelancers en Chile (Persona Natural):**

1. **Inscripción en SII:**
   - Inicio de Actividades en https://www.sii.cl
   - Código de Actividad: 620200 (Actividades de consultorías informáticas)

2. **Emisión de Boletas Electrónicas:**
   - Usar portal MIPYME del SII
   - Retención automática del 11.5% (Impuesto Segunda Categoría)

3. **Declaración Anual (Abril):**
   - Formulario 22 (Renta)
   - Gastos deducibles: computador, internet, software

**Ejemplo de Boleta:**
```
BOLETA DE HONORARIOS ELECTRÓNICA N° 123456

RECEPTOR:
[Nombre del cliente]
RUT: XX.XXX.XXX-X

DETALLE:
Desarrollo de Sistema E-commerce
Hito 2: Frontend + Backend Funcional

MONTO BRUTO:     $2,000,000 CLP
RETENCIÓN 11.5%:   $230,000 CLP
LÍQUIDO A PAGAR: $1,770,000 CLP
```

### 10.2 Si Operas con Empresa (SPA/EIRL)

**Ventajas:**
- Factura con IVA (19%) recuperable para clientes con giro comercial
- Proyección más profesional
- Responsabilidad limitada

**Desventajas:**
- Costos de constitución ($50,000 - $150,000 CLP)
- Contabilidad obligatoria (contador externo)
- Declaraciones mensuales (F29)

---

## RESUMEN EJECUTIVO PARA EL AGENTE IA

**Cuando el supervisor te pida generar un contrato:**

1. Usa las plantillas de este documento
2. Personaliza con datos del proyecto
3. SIEMPRE incluye:
   - Cláusula de IP
   - Cláusula de código generado por IA
   - NDA si hay datos sensibles
   - Garantía de 60 días
   - Protección contra scope creep
4. Genera en formato DOCX/PDF profesional
5. Recuérdale al supervisor que lo revise con un abogado antes de firmar

**Herramientas:**
- Usa `docx` skill para generar contratos en Word
- Usa `pdf` skill para exportar a PDF firmable

---

**FIN DE DOCUMENTO LEGAL**

Este documento protege al freelancer y al cliente. NO es un sustituto de asesoría legal profesional.
