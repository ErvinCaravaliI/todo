### Aplicación del método de Newton–Raphson — Informe técnico

- **Autores**: Ervin Caravali Ibarra (1925648), Juan Esteban Ortiz Bejarano (2410227), Brayan Camilo Urrea Jurado (2410023)

### Propósito del sistema
- Resolver un sistema de ecuaciones no lineales para el campo de velocidades en un dominio discretizado, empleando **Newton–Raphson** hasta alcanzar equilibrio (residuales cercanos a cero).

### Paso a paso detallado (basado en las figuras del avance)
1) Definición del dominio y malla
   - Malla total: 5 × 50 celdas con paso h = 8.
   - Puntos internos de fluido: j = 1..3 y i = 1..48 (se excluyen fronteras externas).
   - Viga central inferior: en la fila j = 1, columnas i = 20..29; esos 10 nodos son fijos (sólido) y no forman parte de las incógnitas.
   - Total de incógnitas: 134 nodos de velocidad V a resolver.
   - Referencia visual: figuras de malla y dominio (ver imágenes del avance).

2) De ecuación de relajación a ecuación de equilibrio
   - La ecuación base se reescribe como F(V) = 0 para imponer equilibrio en cada nodo de incógnita.
   - El término convectivo se incluye para capturar asimetrías en i − 1 e i + 1.
   - Objetivo: que F(V) → 0 en toda la región de incógnitas.

3) Vector de incógnitas V y vector de residuales F
   - Se construye V apilando las velocidades de cada nodo interno según un mapeo (i, j) → índice plano m ∈ [0, 133].
   - F tiene la misma longitud que V; cada entrada F[m] es el residual de la ecuación en el nodo correspondiente.
   - El mapeo excluye automáticamente nodos fijos (fronteras y viga) y aplica desplazamientos por filas para mantener índices contiguos.

4) Ensamblaje de F(V)
   - Para cada nodo (i, j) incógnita, se evalúa V en el nodo central y en sus 4 vecinos (i ± 1, j) y (i, j ± 1).
   - Residual típico (esquema de 5 puntos con convección):
     - Término difusivo: 4·V(i,j) − V(i+1,j) − V(i−1,j) − V(i,j+1) − V(i,j−1).
     - Término convectivo: FACTOR_CONVECCION · V(i,j) · [V(i−1,j) − V(i+1,j)].
   - Se arma F[m] combinando ambos términos con los valores fijos cuando el vecino está en frontera o viga.

5) Ensamblaje del Jacobiano J(V)
   - Para cada ecuación (nodo m), se calculan derivadas parciales respecto a:
     - Nodo central m: dF/dV(i,j) = 4 + FACTOR_CONVECCION·V(i−1,j) − FACTOR_CONVECCION·V(i+1,j).
     - Vecino derecho (i+1,j): −1 − FACTOR_CONVECCION·V(i,j), si es incógnita.
     - Vecino izquierdo (i−1,j): −1 + FACTOR_CONVECCION·V(i,j), si es incógnita.
     - Vecino superior (i,j+1): −1, si es incógnita.
     - Vecino inferior (i,j−1): −1, si es incógnita.
   - Los coeficientes se colocan en las columnas correspondientes a los índices de esos vecinos; si un vecino es fijo, no genera columna pero sí aporta al término independiente de F.

6) Sistema lineal y actualización
   - En la iteración k: J(Vᵏ) · ΔV = −F(Vᵏ).
   - Se resuelve el sistema esparso (por ejemplo, con `scipy.sparse.linalg.spsolve`).
   - Se actualiza Vᵏ⁺¹ = Vᵏ + ΔV.

7) Condiciones de frontera y sólidos
   - Suelo (j = 0): V = 0.0.
   - Superficie G (j = 4): V = V0.
   - Entrada F (i = 0) y salida H (i = 49): V = V0.
   - Viga (j = 1, i = 20..29): V = 0.0 (nodos omitidos del vector de incógnitas).

8) Criterios de convergencia y control
   - Tolerancia TOL sobre ||F|| (norma euclídea) y, de forma adicional, sobre ||ΔV|| para detectar correcciones insignificantes.
   - Límite de iteraciones MAX_ITER.
   - Registro por iteración: norma del residual y mensajes de convergencia/advertencia.

### Parámetros principales
- `V0 = 1.0`
- `FACTOR_CONVECCION = 4.0`
- `MAX_ITER = 100`
- `TOL = 1e-6`
- Tamaño de malla interna: `J_MAX = 3`, `I_MAX = 48`
- `N_INCÓGNITAS = 134`

### Funciones y estructura sugerida de implementación
- `get_velocidad_fija(i, j)`: devuelve valores fijos en fronteras y viga.
- `map_to_index(i, j)`: mapea (i, j) a índice plano, gestionando omisiones y desplazamientos.
- `get_V_value(i, j, V_k)`: obtiene V(i, j) desde V_k o desde las condiciones fijas.
- `ensamblar_FJ(V_k)`: construye F y J considerando nodo central y vecinos.
- `solve_newton_raphson()`: bucle de iteración, resolución del sistema y actualización.

### Recomendaciones prácticas
- Extraer el código del avance a un módulo Python y añadir pruebas sobre el mapeo de índices y los contornos.
- Inspeccionar la estructura de J (dispersión y condicionamiento) y, si es necesario, emplear precondicionadores.
- Exportar resultados y visualizar perfiles/secciones para validar el comportamiento cercano a la viga y a las fronteras.

