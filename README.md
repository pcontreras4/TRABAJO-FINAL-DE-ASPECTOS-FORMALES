# Inventario farmacéutico: especificación formal en VDM++ y model checking

Proyecto final del curso **Aspectos Formales de Especificación y Verificación**
Escuela Profesional de Ingeniería de Sistemas – Universidad Nacional de San Agustín de Arequipa

Especificamos en VDM++ el subsistema de control de inventario para la dispensación segura de medicamentos y verificamos el ciclo de vida de un lote con model checking (NuSMV y UPPAAL). La regla central es que **nunca se dispensa de un lote vencido, sin stock suficiente o por un responsable no autorizado**.

## Estructura

```
README.md
01_especificacion_vdm/
├── Inventario.vpp              # Tipos, estado, invariantes (I1-I3) y operaciones con pre/post
└── TestInventario.vpp          # Casos positivos (T1-T5) y negativos (N1-N7)
02_model_checking/
├── nusmv/
│   └── medicamento.smv         # Sistema de transición y propiedades LTL (R1-R5)
└── uppaal/
    ├── lote.xml                # Autómata temporizado y consultas TCTL (U1-U5)
    └── lote_control.xml        # Prueba de control: sin la transición a Vencido
03_resultados/
└── resultados_validacion.txt   # Resultados de VDM++, NuSMV y UPPAAL
```

## Cómo ejecutar

### 1. VDM++ (VDMTools o VDMJ)

1. Cargar `Inventario.vpp` y `TestInventario.vpp` en el mismo proyecto.
2. Activar la comprobación dinámica de tipos, invariantes, precondiciones y postcondiciones.
3. En el intérprete:
   ```
   create t := new TestInventario()
   print t.Ejecutar()
   print t.N1()
   ```
   `Ejecutar()` devuelve los cinco casos positivos con `true`. Cada caso negativo (`N1` a `N7`) se ejecuta por separado y debe terminar en error.

### 2. NuSMV

```
NuSMV medicamento.smv
```

Para la prueba de control, borrar en `medicamento.smv` la línea `(estado = disponible & dias = 1) : vencido;` y volver a ejecutar.

### 3. UPPAAL

Abrir `lote.xml` en UPPAAL, ir a la pestaña **Verifier** y pulsar **Check**. También se puede usar la línea de comandos:

```
verifyta lote.xml
verifyta lote_control.xml
```

## Resultados

| Herramienta | Resultado |
|---|---|
| VDM++ – casos positivos | 5 de 5 devuelven `true` |
| VDM++ – casos negativos | 7 de 7 rechazados por la precondición o invariante esperada |
| NuSMV – propiedades R1 a R5 | 5 de 5 verdaderas |
| UPPAAL – consultas U1 a U5 | 5 de 5 satisfechas |

**Prueba de control:** al quitar la transición a Vencido, R4 y R5 (NuSMV) y U4 y U5 (UPPAAL) fallan. Así se comprueba que los verificadores detectan errores y no aprueban todo por defecto.

El detalle completo está en `03_resultados/resultados_validacion.txt`.

## Propiedades verificadas

| Id | NuSMV (LTL) | UPPAAL (TCTL) | Qué garantiza |
|---|---|---|---|
| R1 / U1 | `G (estado = agotado -> stock = 0)` | `A[] (Agotado imply stock == 0)` | Agotado significa sin unidades |
| R2 / U2 | `G (estado = disponible -> stock > 0)` | `A[] (Disponible imply stock > 0)` | Un lote disponible tiene unidades |
| R3 / U3 | `G (estado = vencido -> stock > 0)` | `A[] (Vencido imply stock > 0)` | Un vencido conserva unidades |
| R4 / U4 | `F (estado = agotado \| estado = vencido)` | `A<> (Agotado or Vencido)` | Todo lote termina su ciclo |
| R5 / U5 | `G (estado = disponible -> dias > 0)` | `A[] (Disponible imply t <= VIGENCIA)` | Solo se dispensa de un lote no vencido |

## Autores

- Contreras Huamani, Paul Michaell
- Choque Sánchez, Alejandra Camila
- Ccahua Chise, Elizabeth Karina
- Llacma Quispe, Kevin Andree

**Docente:** Mg. Maribel Molina Barriga
