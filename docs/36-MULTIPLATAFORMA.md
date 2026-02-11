# 36 - SISTEMAS MULTIPLATAFORMA (WEB, MÓVIL, DESKTOP)

> "Write once, run everywhere" es un sueño. La realidad es "Write carefully, run almost everywhere".  
> Cada plataforma tiene sus peculiaridades. Elige la herramienta correcta para el job.

---

## SECCION 1: MATRIZ DE DECISIÓN

### 1.1 Comparativa de Tecnologías

| Tecnología | Plataformas | Lenguaje | Performance | Tamaño Binario | Curva Aprendizaje | Ecosistema |
|------------|-------------|----------|-------------|----------------|-------------------|------------|
| **Flutter** | Web, iOS, Android, Windows, macOS, Linux | Dart | Excelente (Skia) | ~15-20 MB | Media | Creciente |
| **React Native** | iOS, Android, Web* | JS/TS | Bueno (JSC/Hermes) | ~10-15 MB | Media-Baja | Maduro |
| **Tauri** | Windows, macOS, Linux | Rust + HTML/CSS/JS | Excelente | ~5-10 MB | Alta (Rust) | Emergente |
| **Electron** | Windows, macOS, Linux | JS/TS | Medio (Chromium) | ~80-120 MB | Baja | Muy Maduro |
| **Kotlin Multiplatform** | iOS, Android, Desktop, Web | Kotlin | Excelente (Nativo) | Varía | Alta | Emergente |
| **Rust + GTK4** | Linux, Windows, macOS | Rust | Excelente | ~5-8 MB | Muy Alta | Nicho |
| **Compose Multiplatform** | Android, iOS, Desktop, Web | Kotlin | Excelente | Varía | Media-Alta | Creciente |

*React Native Web requiere configuración adicional

### 1.2 Recomendaciones por Caso de Uso

#### A) Solo Desktop (Windows/macOS/Linux)

**Opción 1 (Recomendada):** **Tauri + Vue 3 / React**
```
Ventajas:
- Binarios pequeños (5-10 MB)
- Excelente rendimiento
- Seguro (Rust backend)
- Actualizaciones automáticas integradas

Desventajas:
- Rust tiene curva de aprendizaje alta
- No soporta móvil

Casos de uso ideales:
- Editores de texto/código
- Herramientas de productividad
- Clientes de bases de datos
- Dashboards empresariales
```

**Opción 2:** **Electron + Vue 3 / React**
```
Ventajas:
- Ecosistema maduro
- Fácil para devs web
- Integración con Node.js

Desventajas:
- Binarios pesados (80-120 MB)
- Mayor consumo de RAM

Casos de uso ideales:
- Si ya tienes backend Node.js
- Si el equipo NO sabe Rust
- Apps que requieren acceso a paquetes npm nativos
```

**Opción 3:** **Rust + GTK4** (Solo para nativos puros)
```
Ventajas:
- Máximo rendimiento
- Binarios ultra pequeños
- Integración profunda con el OS

Desventajas:
- Curva de aprendizaje MUY alta
- UI menos moderna que Flutter/Tauri
- Código más verboso

Casos de uso ideales:
- Herramientas de sistema
- Apps de alto rendimiento (edición video, audio)
- Solo si el equipo domina Rust
```

#### B) Solo Móvil (iOS + Android)

**Opción 1 (Recomendada):** **Flutter**
```
Ventajas:
- UI nativa hermosa
- Hot reload rápido
- Widget system rico
- Rendimiento cercano a nativo

Desventajas:
- Dart es un lenguaje nuevo para muchos
- Comunidad menor que React Native

Casos de uso ideales:
- Apps con UI compleja
- Apps con animaciones
- Startups que quieren velocidad de desarrollo
```

**Opción 2:** **React Native**
```
Ventajas:
- Ecosistema maduro
- Usa JavaScript/TypeScript (familiar)
- Muchos paquetes de terceros

Desventajas:
- Rendimiento inferior a Flutter/Nativo
- Debugging más complejo
- Puentes JS-Nativo pueden ser un cuello de botella

Casos de uso ideales:
- Equipo ya sabe React
- Necesitas integración con bibliotecas JS existentes
- Prototipado rápido
```

**Opción 3:** **Kotlin Multiplatform (KMP)**
```
Ventajas:
- Compartes lógica de negocio
- UI nativa para cada plataforma (Compose/SwiftUI)
- Máximo rendimiento

Desventajas:
- Curva de aprendizaje alta
- Aún inmaduro (2024)
- Requiere desarrolladores Kotlin/iOS

Casos de uso ideales:
- Apps enterprise con requisitos de rendimiento estrictos
- Equipos que ya tienen devs iOS y Android
- Cuando la UI debe ser 100% nativa
```

#### C) TODO (Web + Móvil + Desktop)

**Opción 1 (Recomendada):** **Flutter**
```
Stack:
- Flutter para TODAS las plataformas
- Backend: NestJS / FastAPI
- BD: PostgreSQL

Ventajas:
- Un solo codebase
- Consistencia de UI en todas las plataformas
- Hot reload en desarrollo

Desventajas:
- Web de Flutter es funcional pero no ideal (SEO limitado)
- Binarios desktop más grandes que Tauri

Casos de uso ideales:
- Dashboards internos
- Apps de productividad
- SaaS con cliente móvil
```

**Opción 2:** **Híbrido (React Native + Tauri + Web tradicional)**
```
Stack:
- Móvil: React Native
- Desktop: Tauri + React
- Web: Nuxt 3 / Next.js
- Backend: NestJS
- Componentes compartidos vía npm package

Ventajas:
- Mejor experiencia en cada plataforma
- SEO óptimo en web
- Binarios desktop pequeños

Desventajas:
- Mayor complejidad de mantenimiento
- Código parcialmente duplicado

Casos de uso ideales:
- Productos con requisitos muy específicos por plataforma
- Cuando SEO es crítico
```

---

## SECCION 2: IMPLEMENTACIÓN POR TECNOLOGÍA

### 2.1 Flutter (Multiplataforma Completa)

#### Instalación y Setup

```bash
# Instalar Flutter
git clone https://github.com/flutter/flutter.git -b stable
export PATH="$PATH:`pwd`/flutter/bin"

# Verificar instalación
flutter doctor

# Crear nuevo proyecto
flutter create mi_app --org com.ejemplo
cd mi_app

# Estructura recomendada
lib/
├── main.dart
├── core/
│   ├── theme/
│   ├── utils/
│   └── constants/
├── features/
│   ├── auth/
│   │   ├── data/
│   │   ├── domain/
│   │   └── presentation/
│   └── products/
│       ├── data/
│       ├── domain/
│       └── presentation/
└── shared/
    ├── widgets/
    └── models/
```

#### Ejemplo: Screen con Null Safety

```dart
// lib/features/products/presentation/product_list_screen.dart

import 'package:flutter/material.dart';

class ProductListScreen extends StatefulWidget {
  const ProductListScreen({Key? key}) : super(key: key);

  @override
  State<ProductListScreen> createState() => _ProductListScreenState();
}

class _ProductListScreenState extends State<ProductListScreen> {
  List<Product> _products = [];
  bool _isLoading = true;

  @override
  void initState() {
    super.initState();
    _loadProducts();
  }

  Future<void> _loadProducts() async {
    setState(() => _isLoading = true);
    
    try {
      final products = await ProductRepository().getAll();
      setState(() {
        _products = products;
        _isLoading = false;
      });
    } catch (e) {
      setState(() => _isLoading = false);
      ScaffoldMessenger.of(context).showSnackBar(
        SnackBar(content: Text('Error: $e')),
      );
    }
  }

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(title: const Text('Productos')),
      body: _isLoading
          ? const Center(child: CircularProgressIndicator())
          : ListView.builder(
              itemCount: _products.length,
              itemBuilder: (context, index) {
                final product = _products[index];
                return ListTile(
                  leading: Image.network(product.imageUrl),
                  title: Text(product.name),
                  subtitle: Text('\$${product.price.toStringAsFixed(2)}'),
                  onTap: () => _showProductDetails(product),
                );
              },
            ),
      floatingActionButton: FloatingActionButton(
        onPressed: _loadProducts,
        child: const Icon(Icons.refresh),
      ),
    );
  }

  void _showProductDetails(Product product) {
    Navigator.push(
      context,
      MaterialPageRoute(
        builder: (_) => ProductDetailScreen(product: product),
      ),
    );
  }
}
```

#### Builds por Plataforma

```bash
# Android
flutter build apk --release  # APK
flutter build appbundle      # AAB para Play Store

# iOS (requiere macOS + Xcode)
flutter build ios --release
flutter build ipa             # Para App Store

# Web
flutter build web --release

# Desktop
flutter build windows --release  # Windows
flutter build macos --release    # macOS
flutter build linux --release    # Linux
```

### 2.2 Tauri (Desktop Eficiente)

#### Instalación y Setup

```bash
# Prerrequisitos (Ubuntu/Debian)
sudo apt install -y libwebkit2gtk-4.0-dev \
    build-essential \
    curl \
    wget \
    libssl-dev \
    libgtk-3-dev \
    libayatana-appindicator3-dev \
    librsvg2-dev

# Instalar Rust
curl --proto '=https' --tlsv1.2 -sSf https://sh.rustup.rs | sh

# Crear proyecto Tauri
npm create tauri-app@latest

# Estructura
src-tauri/
├── Cargo.toml          # Dependencias Rust
├── src/
│   ├── main.rs         # Backend Rust
│   └── commands.rs     # Comandos expuestos a frontend
└── icons/

src/                    # Frontend (Vue/React)
├── components/
├── views/
└── App.vue
```

#### Ejemplo: Comando Rust Expuesto a Frontend

```rust
// src-tauri/src/commands.rs

use tauri::command;
use serde::{Deserialize, Serialize};

#[derive(Debug, Serialize, Deserialize)]
pub struct Product {
    pub id: i32,
    pub name: String,
    pub price: f64,
}

#[command]
pub fn get_products() -> Result<Vec<Product>, String> {
    // Lógica de BD o API
    Ok(vec![
        Product { id: 1, name: "Laptop".to_string(), price: 999.99 },
        Product { id: 2, name: "Mouse".to_string(), price: 29.99 },
    ])
}

#[command]
pub async fn save_product(product: Product) -> Result<String, String> {
    // Validación
    if product.name.is_empty() {
        return Err("El nombre no puede estar vacío".to_string());
    }
    
    // Guardar en BD (ejemplo con SQLite)
    match db::insert_product(&product).await {
        Ok(_) => Ok("Producto guardado correctamente".to_string()),
        Err(e) => Err(format!("Error al guardar: {}", e)),
    }
}
```

```rust
// src-tauri/src/main.rs

#![cfg_attr(not(debug_assertions), windows_subsystem = "windows")]

mod commands;

fn main() {
    tauri::Builder::default()
        .invoke_handler(tauri::generate_handler![
            commands::get_products,
            commands::save_product
        ])
        .run(tauri::generate_context!())
        .expect("Error al correr la aplicación");
}
```

```typescript
// Frontend (Vue 3): src/components/ProductList.vue

<script setup lang="ts">
import { ref, onMounted } from 'vue'
import { invoke } from '@tauri-apps/api/tauri'

interface Product {
  id: number
  name: string
  price: number
}

const products = ref<Product[]>([])
const isLoading = ref(true)

onMounted(async () => {
  try {
    products.value = await invoke<Product[]>('get_products')
  } catch (error) {
    console.error('Error loading products:', error)
  } finally {
    isLoading.value = false
  }
})

const handleSave = async (product: Product) => {
  try {
    const message = await invoke<string>('save_product', { product })
    alert(message)
  } catch (error) {
    alert(`Error: ${error}`)
  }
}
</script>

<template>
  <div v-if="isLoading">Cargando...</div>
  <ul v-else>
    <li v-for="product in products" :key="product.id">
      {{ product.name }} - ${{ product.price }}
    </li>
  </ul>
</template>
```

#### Builds

```bash
# Desarrollo
npm run tauri dev

# Producción
npm run tauri build

# Genera instaladores nativos:
# - Windows: .exe, .msi
# - macOS: .dmg, .app
# - Linux: .deb, .AppImage
```

### 2.3 Electron (Desktop con Node.js)

#### Setup Básico

```bash
npm init
npm install --save-dev electron

# Estructura
my-app/
├── main.js           # Proceso principal (Node.js)
├── preload.js        # Bridge seguro
├── renderer/
│   ├── index.html
│   └── app.js        # Proceso renderer (Frontend)
└── package.json
```

#### Ejemplo: main.js

```javascript
// main.js
const { app, BrowserWindow, ipcMain } = require('electron')
const path = require('path')

let mainWindow

function createWindow() {
  mainWindow = new BrowserWindow({
    width: 1200,
    height: 800,
    webPreferences: {
      preload: path.join(__dirname, 'preload.js'),
      contextIsolation: true,
      nodeIntegration: false // SEGURIDAD
    }
  })

  mainWindow.loadFile('renderer/index.html')
  
  // DevTools en desarrollo
  if (!app.isPackaged) {
    mainWindow.webContents.openDevTools()
  }
}

app.whenReady().then(createWindow)

// IPC Handler (Backend Node.js)
ipcMain.handle('get-products', async () => {
  // Lógica de BD
  return [
    { id: 1, name: 'Laptop', price: 999.99 },
    { id: 2, name: 'Mouse', price: 29.99 }
  ]
})
```

```javascript
// preload.js (Bridge Seguro)
const { contextBridge, ipcRenderer } = require('electron')

contextBridge.exposeInMainWorld('electronAPI', {
  getProducts: () => ipcRenderer.invoke('get-products'),
  saveProduct: (product) => ipcRenderer.invoke('save-product', product)
})
```

```html
<!-- renderer/index.html -->
<!DOCTYPE html>
<html>
<head>
  <title>Mi App</title>
</head>
<body>
  <div id="app">
    <h1>Productos</h1>
    <ul id="product-list"></ul>
  </div>
  
  <script>
    async function loadProducts() {
      const products = await window.electronAPI.getProducts()
      const list = document.getElementById('product-list')
      list.innerHTML = products.map(p => 
        `<li>${p.name} - $${p.price}</li>`
      ).join('')
    }
    
    loadProducts()
  </script>
</body>
</html>
```

---

## SECCION 3: INTEGRACIÓN CON HARDWARE (POS)

### 3.1 Pistola de Código de Barras (USB Serial)

```typescript
// Tauri: src-tauri/src/barcode.rs

use serialport::{SerialPort, SerialPortType};
use std::time::Duration;

#[command]
pub fn scan_barcode() -> Result<String, String> {
    let ports = serialport::available_ports()
        .map_err(|e| format!("Error al listar puertos: {}", e))?;
    
    // Buscar pistola de código de barras (ejemplo: VID/PID conocido)
    let barcode_port = ports.iter().find(|p| {
        if let SerialPortType::UsbPort(info) = &p.port_type {
            info.vid == 0x0C2E && info.pid == 0x0B61 // Ejemplo Honeywell
        } else {
            false
        }
    });
    
    match barcode_port {
        Some(port_info) => {
            let mut port = serialport::new(&port_info.port_name, 9600)
                .timeout(Duration::from_millis(1000))
                .open()
                .map_err(|e| format!("Error al abrir puerto: {}", e))?;
            
            let mut buffer: Vec<u8> = vec![0; 128];
            match port.read(&mut buffer) {
                Ok(n) => {
                    let barcode = String::from_utf8_lossy(&buffer[..n]);
                    Ok(barcode.trim().to_string())
                },
                Err(e) => Err(format!("Error al leer: {}", e))
            }
        },
        None => Err("No se encontró pistola de código de barras".to_string())
    }
}
```

### 3.2 Impresora Térmica (ESCPOS)

```typescript
// Electron: main.js

const escpos = require('escpos')
escpos.USB = require('escpos-usb')

ipcMain.handle('print-receipt', async (event, order) => {
  try {
    const device = new escpos.USB()
    const printer = new escpos.Printer(device)
    
    device.open(function(error) {
      if (error) throw error
      
      printer
        .font('a')
        .align('ct')
        .style('bu')
        .size(2, 2)
        .text('TICKET DE VENTA')
        .style('normal')
        .size(1, 1)
        .text('--------------------------------')
        .align('lt')
        .text(`Orden #${order.id}`)
        .text(`Fecha: ${new Date().toLocaleDateString()}`)
        .text('--------------------------------')
        
      // Items
      order.items.forEach(item => {
        printer.text(`${item.name}`)
        printer.text(`  ${item.quantity} x $${item.price} = $${item.total}`)
      })
      
      printer
        .text('--------------------------------')
        .align('rt')
        .style('b')
        .text(`TOTAL: $${order.total}`)
        .style('normal')
        .cut()
        .close()
    })
    
    return { success: true }
  } catch (error) {
    return { success: false, error: error.message }
  }
})
```

---

## SECCION 4: DISTRIBUCIÓN Y ACTUALIZACIONES

### 4.1 Tauri Updater (Auto-Actualizaciones)

```rust
// src-tauri/tauri.conf.json

{
  "tauri": {
    "updater": {
      "active": true,
      "endpoints": [
        "https://mi-servidor.com/releases/{{target}}/{{current_version}}"
      ],
      "dialog": true,
      "pubkey": "dW50cnVzdGVkIGNvbW1lbnQ6IG1pbmlzaWduIHB1YmxpYyBrZXk6..."
    }
  }
}
```

```rust
// src-tauri/src/main.rs

use tauri::api::updater::check_update;

fn main() {
    tauri::Builder::default()
        .setup(|app| {
            // Check updates on startup
            let handle = app.handle();
            tauri::async_runtime::spawn(async move {
                match check_update(handle).await {
                    Ok(update) => {
                        if update.is_update_available() {
                            update.download_and_install().await.ok();
                        }
                    }
                    Err(e) => println!("Failed to check for updates: {}", e),
                }
            });
            Ok(())
        })
        .run(tauri::generate_context!())
        .expect("error while running tauri application");
}
```

### 4.2 Firma de Binarios (Windows)

```powershell
# Generar certificado self-signed (desarrollo)
New-SelfSignedCertificate -Type CodeSigningCert -Subject "CN=MiEmpresa" -CertStoreLocation Cert:\CurrentUser\My

# Firmar .exe
signtool sign /fd SHA256 /a "mi-app.exe"

# Para producción, compra un certificado de Digicert/Sectigo
```

---

## RESUMEN PARA EL AGENTE IA

**Cuando el supervisor solicite una app multiplataforma:**

1. **Pregunta el alcance exacto:**
   - ¿Solo desktop, solo móvil, o todo?
   - ¿Qué plataformas son prioritarias?

2. **Recomienda el stack óptimo:**
   - Desktop solo: Tauri (si sabe Rust) o Electron
   - Móvil solo: Flutter (recomendado) o React Native
   - Todo: Flutter (un codebase) o híbrido

3. **Considera hardware:**
   - ¿Necesita imprimir? (Electron + ESCPOS)
   - ¿Scanner de código? (Tauri/Electron + Serial)

4. **Planifica distribución:**
   - Instaladores nativos (.exe, .dmg, .deb)
   - Auto-actualizaciones (Tauri Updater)
   - Firma de código (Windows Code Signing)

**Stacks Recomendados:**

```
POS (Desktop + Hardware):
- Electron + Vue 3 + NestJS + SQLite

Dashboard Interno (Web + Desktop):
- Flutter (todas las plataformas)

App Móvil Pública:
- Flutter (iOS + Android + Web)

Herramienta CLI/Desktop Ligera:
- Tauri + Vue 3
```

---

**FIN DE DOCUMENTO MULTIPLATAFORMA**
