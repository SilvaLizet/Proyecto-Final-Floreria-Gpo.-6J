Claro, aquí tienes un ejemplo completo y claro de cómo estructurar una base de datos en **MongoDB** para una florería 🌸. Incluye:

* Base de datos
* Colecciones
* Documentos
* Atributos con tipos de datos

---

## 🌼 1. Crear la Base de Datos

```js
use floreriaAura
```

Esto crea (o selecciona) la base de datos llamada **floreriaAura**.

---

## 🌷 2. Colección: clientes

```js
db.clientes.insertOne({
  nombre: "Ana López",            // string
  correo: "ana@gmail.com",        // string
  telefono: "6561234567",         // string
  direccion: {                    // objeto
    calle: "Av. Flores",          // string
    numero: 123,                  // number
    ciudad: "Juárez",             // string
    codigo_postal: "32000"        // string
  },
  fecha_registro: new Date()      // date
})
```

---

## 🌹 3. Colección: productos (flores/arreglos)

```js
db.productos.insertOne({
  nombre: "Ramo de rosas",        // string
  tipo: "Rosas",                 // string
  precio: 350.50,                // number (decimal)
  stock: 20,                     // number (int)
  descripcion: "Ramo con 12 rosas rojas", // string
  disponible: true               // boolean
})
```

---

## 🌻 4. Colección: pedidos

```js
db.pedidos.insertOne({
  cliente_id: ObjectId(),        // ObjectId (referencia)
  productos: [                   // arreglo de objetos
    {
      producto_id: ObjectId(),   // ObjectId
      cantidad: 2,               // number
      precio_unitario: 350.50    // number
    }
  ],
  total: 701.00,                 // number
  estado: "pendiente",           // string
  fecha_pedido: new Date(),      // date
  metodo_pago: "tarjeta"         // string
})
```

---

## 🌺 5. Colección: empleados

```js
db.empleados.insertOne({
  nombre: "Carlos Pérez",        // string
  puesto: "Vendedor",            // string
  telefono: "6569876543",        // string
  correo: "carlos@floreria.com", // string
  salario: 8000,                 // number
  fecha_contratacion: new Date() // date
})
```

---

## 🌸 Tipos de datos usados

* **string** → texto
* **number** → números (enteros o decimales)
* **boolean** → true / false
* **date** → fechas (`new Date()`)
* **object** → objetos dentro de documentos
* **array** → listas de datos
* **ObjectId** → identificador único (relaciones)

---

## 💡 Nota importante (para tu proyecto)

Este diseño te sirve mucho si estás haciendo una app como tu florería **Aura**, porque:

* Puedes relacionar clientes con pedidos
* Controlar inventario (stock)
* Guardar historial de compras
* Manejar empleados

---

Si quieres, en el siguiente paso te puedo hacer un **`insertMany` con muchos datos (como 20 o 40 registros)** para que lo uses directo en tu proyecto 👀
