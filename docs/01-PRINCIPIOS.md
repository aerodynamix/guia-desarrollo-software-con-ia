# 01 - Principios de Desarrollo

> Estos principios son obligatorios. No son sugerencias.

---

## SOLID

| Principio | Significado | Regla Practica |
|-----------|-------------|----------------|
| **S** - Single Responsibility | Una clase/modulo hace UNA sola cosa | Si describes lo que hace y usas "y", dividelo |
| **O** - Open/Closed | Abierto a extension, cerrado a modificacion | Usa interfaces y composicion, no modifiques codigo existente |
| **L** - Liskov Substitution | Las subclases deben ser intercambiables | Si heredas, no cambies el comportamiento esperado |
| **I** - Interface Segregation | Interfaces pequenas y especificas | No obligues a implementar metodos que no se usan |
| **D** - Dependency Inversion | Depende de abstracciones, no de implementaciones | Inyecta dependencias, no las instancies dentro |

---

## Principios Complementarios

### KISS (Keep It Simple, Stupid)
- La solucion mas simple que funcione es la correcta.
- No uses patrones de diseno si no hay un problema real que resolver.
- Si un `if` resuelve el problema, no crees una Strategy.

### DRY (Don't Repeat Yourself)
- Si copias y pegas codigo, extraelo a una funcion.
- Pero cuidado: duplicar es mejor que una mala abstraccion.
- Regla de tres: si se repite 3 veces, abstrae. Antes no.

### YAGNI (You Aren't Gonna Need It)
- No construyas para el futuro imaginario.
- No agregues configuraciones "por si acaso".
- Resuelve el problema de HOY.

---

## Clean Code (Codigo Limpio)

### Nombrado
```
MALO:  const d = new Date();
BIEN:  const fechaCreacion = new Date();

MALO:  function proc(x, y) {}
BIEN:  function calcularImpuesto(precioBase, tasaImpuesto) {}

MALO:  const arr = [];
BIEN:  const productosCarrito = [];
```

### Funciones
- Maximo 20-30 lineas por funcion.
- Maximo 3 parametros. Si necesitas mas, usa un objeto.
- Una funcion hace UNA cosa. Si tiene "y" en el nombre, dividela.
- Sin efectos secundarios ocultos.

### Archivos
- Maximo 200-300 lineas por archivo.
- Un componente/clase por archivo.
- Nombre del archivo = nombre de lo que exporta.

### Comentarios
- El codigo debe auto-documentarse con buenos nombres.
- Comenta el POR QUE, nunca el QUE.
- No dejes codigo comentado. Si no se usa, se borra.

---

## Patrones de Diseno (Referencia: refactoring.guru)

Usa un patron SOLO cuando el problema lo requiera. No fuerces patrones.

### Creacionales (como se crean objetos)

| Patron | Cuando Usarlo | Ejemplo Real |
|--------|---------------|--------------|
| Factory Method | Crear objetos sin especificar clase exacta | Crear diferentes tipos de notificacion (email, SMS, push) |
| Builder | Construir objetos complejos paso a paso | Armar queries SQL o formularios dinamicos |
| Singleton | Una sola instancia global | Conexion a BD, configuracion de app |

### Estructurales (como se componen objetos)

| Patron | Cuando Usarlo | Ejemplo Real |
|--------|---------------|--------------|
| Adapter | Hacer compatible una interfaz con otra | Integrar pasarela de pago externa |
| Facade | Simplificar una interfaz compleja | Wrapper para multiples servicios de envio |
| Decorator | Agregar comportamiento sin modificar clase | Middleware de autenticacion + logging |
| Composite | Estructuras tipo arbol | Menu con submenus, categorias anidadas |

### Comportamiento (como se comunican objetos)

| Patron | Cuando Usarlo | Ejemplo Real |
|--------|---------------|--------------|
| Observer | Reaccionar a cambios de estado | Notificar UI cuando cambia el carrito |
| Strategy | Intercambiar algoritmos en runtime | Diferentes metodos de calculo de envio |
| State | Comportamiento depende del estado | Estados de un pedido (pendiente, pagado, enviado) |
| Command | Encapsular acciones como objetos | Sistema de undo/redo, cola de tareas |

---

## Code Smells (Codigo que Huele Mal)

Detecta y corrige estos problemas:

```
SMELL                    | SOLUCION
-------------------------|----------------------------------
Funcion muy larga        | Extraer en funciones pequenas
Clase muy grande         | Dividir en clases con SRP
Codigo duplicado         | Extraer a funcion/modulo comun
Parametros excesivos     | Usar objeto de parametros
Cadena de ifs/switch     | Usar Strategy o polimorfismo
Feature Envy             | Mover metodo a la clase correcta
Comentarios excesivos    | Mejorar nombres de variables
Variables temporales     | Extraer a metodo con nombre claro
Herencia profunda        | Preferir composicion
Codigo muerto            | Eliminar sin piedad
```

---

## Prevención de Deuda Técnica

### 1. Regla del Boy Scout
> *"Deja el código más limpio de lo que lo encontraste."*
Si tocas un archivo para añadir una feature, aprovecha de renombrar esa variable confusa o extraer esa función gigante. Pequeñas mejoras constantes evitan la reescritura total.

### 2. Complejidad Ciclomática (KISS)
Evita el anidamiento profundo ("Arrow Code").
```javascript
// MAL
if (usuario) {
  if (usuario.activo) {
    if (saldo > 0) {
      procesar();
    }
  }
}

// BIEN (Guard Clauses)
if (!usuario || !usuario.activo || saldo <= 0) return;
procesar();
```

### 3. Comentarios Estratégicos
El código dice **QUÉ** se hace. Los comentarios dicen **POR QUÉ**.
*   **Mal:** `// Incrementa i en 1` (Obvio)
*   **Bien:** `// Se incrementa en 1 para compensar el índice base-0 de la API externa` (Contexto valioso)
