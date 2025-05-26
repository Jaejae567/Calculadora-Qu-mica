<!DOCTYPE html>
<html lang="es">
<head>
<meta charset="UTF-8" />
<meta name="viewport" content="width=device-width, initial-scale=1" />
<title>Calculadora Estequiométrica y Balanceadora</title>
<style>
  body { font-family: Arial, sans-serif; max-width: 600px; margin: 20px auto; background: #f0f0f0; }
  h1 { text-align: center; }
  label { display: block; margin-top: 15px; font-weight: bold; }
  input, select, button { width: 100%; padding: 8px; margin-top: 5px; font-size: 1em; border-radius: 4px; border: 1px solid #ccc; }
  button { background: #007bff; color: white; border: none; cursor: pointer; margin-top: 20px; }
  button:hover { background: #0056b3; }
  #resultado { white-space: pre-wrap; margin-top: 20px; background: white; padding: 15px; border-radius: 6px; box-shadow: 0 0 10px rgba(0,0,0,0.1);}
</style>
</head>
<body>

<h1>Calculadora Estequiométrica y Balanceadora</h1>

<label for="ecuacion">Ecuación química (sin balancear):</label>
<input type="text" id="ecuacion" placeholder="Ej: H2 + O2 -> H2O" />

<label for="compuesto">Compuesto conocido:</label>
<input type="text" id="compuesto" placeholder="Ej: H2" />

<label for="tipoCantidad">Tipo de cantidad conocida:</label>
<select id="tipoCantidad">
  <option value="masa">Masa (g)</option>
  <option value="moles">Moles (mol)</option>
  <option value="volumen">Volumen (L, gases a CNPT)</option>
</select>

<label for="cantidad">Cantidad conocida:</label>
<input type="number" id="cantidad" placeholder="Ej: 4.04" min="0" step="any" />

<button onclick="calcular()">Calcular</button>

<div id="resultado"></div>

<script>
const R = 0.0821; 
const T = 298; 
const P = 1;

const masasMolares = {
  "H": 1.008,
  "O": 15.999,
  "C": 12.011,
  "N": 14.007,
  "Cl": 35.45,
  "Na": 22.99,
  "H2": 2.016,
  "O2": 31.998,
  "H2O": 18.015,
  "CO2": 44.01,
  "CH4": 16.04,
  "NaCl": 58.44
};

function calcular() {
  const ecuacionInput = document.getElementById("ecuacion").value.trim();
  const compuesto = document.getElementById("compuesto").value.trim();
  const tipoCantidad = document.getElementById("tipoCantidad").value;
  const cantidadStr = document.getElementById("cantidad").value;

  const resultadoDiv = document.getElementById("resultado");
  resultadoDiv.textContent = "";

  if (!ecuacionInput.includes("->")) {
    alert("Por favor, ingresa una ecuación válida con '->'");
    return;
  }
  if (!compuesto) {
    alert("Ingresa un compuesto conocido");
    return;
  }
  if (!cantidadStr || isNaN(cantidadStr) || Number(cantidadStr) <= 0) {
    alert("Ingresa una cantidad válida mayor que cero");
    return;
  }
  let cantidad = Number(cantidadStr);

  let lados = ecuacionInput.split("->");
  if (lados.length !== 2) {
    alert("Formato de ecuación inválido.");
    return;
  }
  let reactivosRaw = lados[0].split("+").map(s => s.trim());
  let productosRaw = lados[1].split("+").map(s => s.trim());

  let { coefReactivos, coefProductos, elementos } = balancearEcuacion(reactivosRaw, productosRaw);
  if (!coefReactivos || !coefProductos) {
    resultadoDiv.textContent = "No se pudo balancear la ecuación con este método básico.";
    return;
  }

  let ecuacionBalanceada = "";
  ecuacionBalanceada += coefReactivos.map((c, i) => (c > 1 ? c : "") + reactivosRaw[i]).join(" + ");
  ecuacionBalanceada += " -> ";
  ecuacionBalanceada += coefProductos.map((c, i) => (c > 1 ? c : "") + productosRaw[i]).join(" + ");

  let masasCompuestos = {};
  for (let c of [...reactivosRaw, ...productosRaw]) {
    masasCompuestos[c] = masaMolarCompuesto(c);
  }

  if (!(compuesto in masasCompuestos)) {
    alert(`No se encontró la masa molar del compuesto conocido: ${compuesto}`);
    return;
  }

  let molesConocidos = 0;
  if (tipoCantidad === "masa") {
    molesConocidos = cantidad / masasCompuestos[compuesto];
  } else if (tipoCantidad === "moles") {
    molesConocidos = cantidad;
  } else if (tipoCantidad === "volumen") {
    // Usamos PV=nRT para gases (asumiendo gas ideal y condiciones normales)
    molesConocidos = (P * cantidad) / (R * T);
  }

  let allCompuestos = [...reactivosRaw, ...productosRaw];
  let allCoeficientes = [...coefReactivos, ...coefProductos];
  let idxCompuesto = allCompuestos.indexOf(compuesto);
  if (idxCompuesto === -1) {
    alert("El compuesto conocido no está en la ecuación.");
    return;
  }
  let coefCompuesto = allCoeficientes[idxCompuesto];

  let factor = molesConocidos / coefCompuesto;

  let textoResultado = `Ecuación balanceada:\n${ecuacionBalanceada}\n\n`;
  textoResultado += `Cantidad conocida:\n- Compuesto: ${compuesto}\n- Moles: ${molesConocidos.toFixed(4)} mol\n\n`;
  textoResultado += `Cálculo de cantidades para todos los compuestos:\n`;

  for (let i = 0; i < allCompuestos.length; i++) {
    let c = allCompuestos[i];
    let coef = allCoeficientes[i];
    let moles = coef * factor;
    let masa = moles * masasCompuestos[c];
    let volumen = (moles * R * T) / P; // en litros, si gas ideal

    textoResultado += `- ${c}:\n`;
    textoResultado += `  Coeficiente: ${coef}\n`;
    textoResultado += `  Moles: ${moles.toFixed(4)} mol\n`;
    textoResultado += `  Masa: ${masa.toFixed(4)} g\n`;
    textoResultado += `  Volumen (gas ideal CNPT): ${volumen.toFixed(4)} L\n\n`;
  }

  resultadoDiv.textContent = textoResultado;
}

function balancearEcuacion(reactivos, productos) {
  // 1) Obtener lista de elementos únicos
  let elementos = new Set();

  function extraerElementos(comp) {
    let regex = /([A-Z][a-z]?)/g;
    let elems = comp.match(regex);
    return elems || [];
  }

  reactivos.forEach(r => extraerElementos(r).forEach(e => elementos.add(e)));
  productos.forEach(p => extraerElementos(p).forEach(e => elementos.add(e)));
  elementos = Array.from(elementos);

  let matriz = [];
  elementos.forEach(elem => {
    let fila = [];
    reactivos.forEach(comp => fila.push(contarElemento(comp, elem)));
    productos.forEach(comp => fila.push(-contarElemento(comp, elem)));
    matriz.push(fila);
  });

  let filas = matriz.length;
  let cols = matriz[0].length;
  let A = matriz.map(r => r.slice());

  for (let i = 0; i < filas; i++) A[i].push(0);
  for (let i = 0; i < filas; i++) {
    A[i][cols] -= A[i][0]; 
    A[i][0] = 0;
  }

  for (let i = 0; i < filas; i++) {
    let pivoteFila = i;
    while (pivoteFila < filas && Math.abs(A[pivoteFila][i+1]) < 1e-10) pivoteFila++;
    if (pivoteFila == filas) continue;
    if (pivoteFila != i) {
      let temp = A[i];
      A[i] = A[pivoteFila];
      A[pivoteFila] = temp;
    }

    let pivote = A[i][i+1];
    if (Math.abs(pivote) < 1e-10) continue;
    for (let j = i; j <= cols; j++) A[i][j] /= pivote;

    for (let k = i+1; k < filas; k++) {
      let factor = A[k][i+1];
      for (let j = i; j <= cols; j++) {
        A[k][j] -= factor * A[i][j];
      }
    }
  }

  let x = new Array(cols).fill(0);
  x[0] = 1; // primer coeficiente fijo

  for (let i = filas -1; i >=0; i--) {
    let sum = A[i][cols];
    for (let j = i+2; j < cols; j++) {
      sum -= A[i][j] * x[j];
    }
    x[i+1] = sum / A[i][i+1];
  }

  let denominadores = x.map(frac => frac.toString().split(".")[1]?.length || 0);
  let maxDecimales = Math.max(...denominadores);
  let factorMult = Math.pow(10, maxDecimales);
  let coeficientes = x.map(n => Math.round(n * factorMult));
  let mcd = coeficientes.reduce((a,b) => gcd(a,b));
  coeficientes = coeficientes.map(c => c / mcd);
  let coefReactivos = coeficientes.slice(0, reactivos.length);
  let coefProductos = coeficientes.slice(reactivos.length);

  return { coefReactivos, coefProductos, elementos };
}

function gcd(a, b) {
  if (!b) return a;
  return gcd(b, a % b);
}

function contarElemento(compuesto, elemento) {
  let regex = new RegExp(elemento + "(\\d*)", "g");
  let total = 0;
  let match;
  while ((match = regex.exec(compuesto)) !== null) {
    total += match[1] ? Number(match[1]) : 1;
  }
  return total;
}

function masaMolarCompuesto(compuesto) {
  let regex = /([A-Z][a-z]?)(\d*)/g;
  let masa = 0;
  let match;
  while ((match = regex.exec(compuesto)) !== null) {
    let elem = match[1];
    let num = match[2] ? Number(match[2]) : 1;
    if (!(elem in masasMolares)) {
      // Intentar con elementos básicos
      masa += 0;
    } else {
      masa += masasMolares[elem] * num;
    }
  }
  return masa;
}

</script>
</body>
</html>
