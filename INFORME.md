### Aplicación del método de Newton–Raphson — Informe del repositorio

- **Autores**: Ervin Caravali Ibarra (1925648), Juan Esteban Ortiz Bejarano (2410227), Brayan Camilo Urrea Jurado (2410023)
- **Fuente de autores**: portada de `Introducción al sistema.pdf`

### Contenido del repositorio
- `Introducción al sistema.pdf`: Documento principal con la portada y contexto del trabajo.
- `AVANCE 2.md`: Desarrollo técnico (explicaciones, fórmulas, imágenes embebidas y código en borrador). Contiene la formulación y el esquema de implementación numérica.
- `AVANCE 2.docx`: Versión en documento editable del avance.

### Objetivo y enfoque del sistema
- **Objetivo**: Resolver un sistema de ecuaciones no lineales derivado de un problema de flujo (velocidad) aplicando **Newton–Raphson** hasta alcanzar equilibrio, i.e., residuales cercanos a cero.
- **Idea clave**: Partir de una ecuación de relajación y reescribirla como **ecuación de equilibrio** F(V) = 0. En cada iteración, se linealiza vía el **Jacobiano** J y se resuelve el sistema lineal para la corrección ΔV.

### Dominio, malla y condiciones
- **Malla**: 5 × 50 celdas (j ∈ [0,4], i ∈ [0,49]). Paso de celda reportado: h = 8.
- **Incógnitas**: Se consideran 3 filas internas (j = 1..3) y 48 columnas internas (i = 1..48) con una viga que fija 10 nodos en j = 1, i = 20..29. Total de incógnitas: **134**.
- **Condiciones de frontera** (valores de velocidad fijos):
  - Suelo (j = 0): 0.0
  - Superficie G (j = 4): V0
  - Entrada F (i = 0): V0
  - Salida H (i = 49): V0
  - Viga central inferior (j = 1, i = 20..29): 0.0

### Formulación de Newton–Raphson
- Se arma el vector de residuales F(V) y la matriz Jacobiana J(V) de tamaño **134 × 134** (una ecuación residual por incógnita).
- En cada iteración k:
  - Se evalúan F(Vᵏ) y J(Vᵏ).
  - Se resuelve el sistema lineal esparso J(Vᵏ) · ΔV = −F(Vᵏ).
  - Se actualiza Vᵏ⁺¹ = Vᵏ + ΔV.
- Criterios de terminación reportados: norma de F(V) < TOL o corrección ||ΔV|| < TOL, con máximo de iteraciones.

### Parámetros principales (según `AVANCE 2.md`)
- `V0 = 1.0`
- `FACTOR_CONVECCION = 4.0`
- `MAX_ITER = 100`
- `TOL = 1e-6`
- Tamaño de malla interna: `J_MAX = 3`, `I_MAX = 48`
- `N_INCÓGNITAS = 134`

### Estructura propuesta del código (resumen de `AVANCE 2.md`)
- Dependencias: `numpy`, `scipy.sparse.lil_matrix`, `scipy.sparse.linalg.spsolve`.
- Funciones clave:
  - `get_velocidad_fija(i, j)`: Devuelve velocidad fija en fronteras y en la viga.
  - `map_to_index(i, j)`: Mapea (i, j) → índice plano en [0, 133], gestionando omisiones por la viga y desplazamientos por filas.
  - `get_V_value(i, j, V_k)`: Devuelve el valor de V en (i, j), usando fijo o el vector de incógnitas.
  - `ensamblar_FJ(V_k)`: Calcula el vector de residuales F y la matriz Jacobiana J esparsa; para cada ecuación, considera nodo central y 4 vecinos, más términos convectivos.
  - `solve_newton_raphson()`: Itera hasta convergencia, resolviendo en cada paso el sistema esparso y actualizando V.

### Consideraciones y limitaciones actuales
- El código aparece como fragmento dentro de `AVANCE 2.md` (no hay archivo `.py` independiente en el repositorio en este momento).
- Las figuras están embebidas como imágenes/base64 dentro del Markdown.
- Para ejecutar, es necesario extraer el código a un archivo Python y contar con `numpy` y `scipy` instalados.

### Instrucciones sugeridas de ejecución
1) Crear un archivo, por ejemplo `solver_newton_raphson.py`, copiando el código de `AVANCE 2.md`.
2) Instalar dependencias (entorno Python 3.x):
   - `pip install numpy scipy`
3) Ejecutar el script:
   - `python solver_newton_raphson.py`
4) Verificar la salida de convergencia (norma del residual por iteración) y el vector final `V`.

### Próximos pasos recomendados
- Separar el código en módulos (`malla.py`, `fronteras.py`, `newton.py`) para mayor claridad.
- Añadir validaciones (dimensiones, índices, límites) y pruebas unitarias.
- Exportar resultados (por ejemplo, `CSV` o `VTK`) y agregar visualización.
- Documentar parámetros de entrada y valores por defecto en un `README.md`.

---
Este informe fue elaborado automáticamente a partir de los archivos presentes en el repositorio e incluye como autores a los indicados en la portada de `Introducción al sistema.pdf`.

