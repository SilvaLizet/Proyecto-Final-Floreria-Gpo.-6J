Esta es una guía estructurada para configurar el entorno de trabajo y desarrollar el proyecto **"ProyectoFloreria"** (enfocado en el CRUD de ramos) utilizando la metodología de agentes especializados.

---

## 1. Estructura de la Habilidad Agente Global `.agents`

Para organizar el flujo de trabajo, crearemos un directorio raíz donde residirán las habilidades de diseño, código y scraping.

**Estructura de Directorios:**
```text
.agents/
├── SKILL.md
├── scripts/        # Automatizaciones de terminal
├── ejemplos/       # Snippets de código de referencia
└── resources/      # Activos visuales y configuración Firebase
```

### Contenido de `SKILL.md`
> El Agente Global coordina tres sub-skills:
> 1. **Diseño:** Crea interfaces en Flutter enfocadas en la experiencia del usuario veterinario/floristería.
> 2. **Código:** Implementa lógica en Dart y conexión con Firestore.
> 3. **Scraping:** (Opcional) Para obtener tendencias de precios o catálogos externos.

---

## 2. Preparación del Entorno (Prerrequisitos)

Antes de codificar, debemos asegurar que el "motor" esté listo. Ejecuta estos comandos en tu terminal de **VS Code** o **Antigravity**:

### Verificación de Herramientas
```bash
# 1. Verificar Flutter
flutter --version

# 2. Instalar FlutterFire CLI (si no lo tienes)
dart pub global activate flutterfire_cli

# 3. Login en Firebase
firebase login
```

### Configuración de Firebase Console
1. Ve a [Firebase Console](https://console.firebase.google.com/).
2. Crea un proyecto llamado **"ProyectoFloreria"**.
3. Habilita **Cloud Firestore** en modo de prueba.
4. En la terminal, dentro de la carpeta del proyecto, vincula Firebase:
   ```bash
   flutterfire configure
   ```

---

## 3. Creación del Proyecto "ProyectoFloreria"

Ejecuta el comando para generar la estructura base:
```bash
flutter create proyectofloreria
cd proyectofloreria
```

### Dependencias en `pubspec.yaml`
Añade estas líneas bajo `dependencies`:
```yaml
dependencies:
  flutter:
    sdk: flutter
  firebase_core: ^latest_version
  cloud_firestore: ^latest_version
  cupertino_icons: ^1.0.2
```
*Ejecuta `flutter pub get` tras guardar.*

---

## 4. Arquitectura de Archivos y Código (CRUD de Ramos)

Organizaremos el código siguiendo una secuencia lógica de capas.

### A. Modelo de Datos (`lib/models/ramo_model.dart`)
```dart
class Ramo {
  String id;
  String nombre;
  double precio;
  String descripcion;

  Ramo({required this.id, required this.nombre, required this.precio, required this.descripcion});

  Map<String, dynamic> toMap() => {
    "nombre": nombre,
    "precio": precio,
    "descripcion": descripcion,
  };

  factory Ramo.fromMap(String id, Map<String, dynamic> map) => Ramo(
    id: id,
    nombre: map["nombre"] ?? '',
    precio: (map["precio"] ?? 0).toDouble(),
    descripcion: map["descripcion"] ?? '',
  );
}
```

### B. Servicio Firestore (`lib/services/firebase_service.dart`)
```dart
import 'package:cloud_firestore/cloud_firestore.dart';
import '../models/ramo_model.dart';

class FirebaseService {
  final CollectionReference _ramosRef = FirebaseFirestore.instance.collection('ramos');

  // Create
  Future<void> addRamo(Ramo ramo) => _ramosRef.add(ramo.toMap());

  // Read
  Stream<List<Ramo>> getRamos() {
    return _ramosRef.snapshots().map((snapshot) =>
        snapshot.docs.map((doc) => Ramo.fromMap(doc.id, doc.data() as Map<String, dynamic>)).toList());
  }

  // Update
  Future<void> updateRamo(Ramo ramo) => _ramosRef.doc(ramo.id).update(ramo.toMap());

  // Delete
  Future<void> deleteRamo(String id) => _ramosRef.doc(id).delete();
}
```

### C. UI: Pantalla Principal y CRUD (`lib/screens/home_screen.dart`)
```dart
import 'package:flutter/material.dart';
import '../services/firebase_service.dart';
import '../models/ramo_model.dart';

class HomeScreen extends StatelessWidget {
  final FirebaseService _service = FirebaseService();

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(title: Text("Florería - Gestión de Ramos")),
      body: StreamBuilder<List<Ramo>>(
        stream: _service.getRamos(),
        builder: (context, snapshot) {
          if (!snapshot.hasData) return Center(child: CircularProgressIndicator());
          final ramos = snapshot.data!;
          return ListView.builder(
            itemCount: ramos.length,
            itemBuilder: (context, index) {
              return ListTile(
                title: Text(ramos[index].nombre),
                subtitle: Text("\$${ramos[index].precio}"),
                trailing: Row(
                  mainAxisSize: MainAxisSize.min,
                  children: [
                    IconButton(icon: Icon(Icons.edit, color: Colors.blue), onPressed: () => _mostrarDialogo(context, ramos[index])),
                    IconButton(icon: Icon(Icons.delete, color: Colors.red), onPressed: () => _service.deleteRamo(ramos[index].id)),
                  ],
                ),
              );
            },
          );
        },
      ),
      floatingActionButton: FloatingActionButton(
        child: Icon(Icons.add),
        onPressed: () => _mostrarDialogo(context),
      ),
    );
  }

  void _mostrarDialogo(BuildContext context, [Ramo? ramo]) {
    final nombreController = TextEditingController(text: ramo?.nombre);
    final precioController = TextEditingController(text: ramo?.precio.toString());

    showDialog(
      context: context,
      builder: (context) => AlertDialog(
        title: Text(ramo == null ? "Nuevo Ramo" : "Editar Ramo"),
        content: Column(
          mainAxisSize: MainAxisSize.min,
          children: [
            TextField(controller: nombreController, decoration: InputDecoration(labelText: "Nombre")),
            TextField(controller: precioController, decoration: InputDecoration(labelText: "Precio"), keyboardType: TextInputType.number),
          ],
        ),
        actions: [
          TextButton(onPressed: () => Navigator.pop(context), child: Text("Cancelar")),
          ElevatedButton(
            onPressed: () {
              final nuevoRamo = Ramo(
                id: ramo?.id ?? '',
                nombre: nombreController.text,
                precio: double.parse(precioController.text),
                descripcion: "Ramo de temporada",
              );
              ramo == null ? _service.addRamo(nuevoRamo) : _service.updateRamo(nuevoRamo);
              Navigator.pop(context);
            },
            child: Text("Guardar"),
          )
        ],
      ),
    );
  }
}
```

---

## 5. Inicialización Principal (`lib/main.dart`)

```dart
import 'package:flutter/material.dart';
import 'package:firebase_core/firebase_core.dart';
import 'screens/home_screen.dart';

void main() async {
  WidgetsFlutterBinding.ensureInitialized();
  await Firebase.initializeApp(); // Requiere flutterfire configure previo
  runApp(MaterialApp(
    debugShowCheckedModeBanner: false,
    theme: ThemeData(primarySwatch: Colors.pink),
    home: HomeScreen(),
  ));
}
```

### Próximos Pasos con “.agents”:
1. **Agente de Diseño:** Mejorar la UI con tarjetas personalizadas (`Card`) e imágenes.
2. **Agente de Código:** Implementar autenticación de usuarios.
3. **Agente de Scraping:** Buscar imágenes de flores en la web para el catálogo.
