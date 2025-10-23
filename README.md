# grupo5cifrado
<!doctype html>
<html lang="es">
<head>
  <meta charset="utf-8" />
  <meta name="viewport" content="width=device-width,initial-scale=1" />
  <title>Cifrado César — Web</title>
  <script src="https://cdn.tailwindcss.com"></script>
</head>
<body class="bg-slate-50 min-h-screen flex items-center justify-center p-6">
  <main class="max-w-4xl w-full bg-white rounded-2xl shadow-lg p-6">
    <header class="mb-4">
      <h1 class="text-2xl font-semibold">Cifrado César — Grupo 5</h1>
      <p class="text-sm text-slate-500">Interfaz web simple de nuestro proyecto, permite al usuario cifrar y descifrar manteniendo mayúsculas y minúsculas, además tiene la opcion de incluir la rotacion de números.</p>
    </header>

    <section class="grid grid-cols-1 md:grid-cols-2 gap-4">
      <div>
        <label class="block text-sm font-medium text-slate-700">Texto</label>
        <textarea id="inputText" rows="8" class="mt-2 p-3 w-full border rounded-md" placeholder="Escribe o pega aquí el texto..."></textarea>

        <div class="mt-3 flex items-center gap-3">
          <label class="text-sm">Desplazamiento</label>
          <input id="shift" type="number" value="3" min="0" class="w-20 p-2 border rounded-md" />

          <label class="flex items-center gap-2 ml-4">
            <input id="rotNumbers" type="checkbox" class="w-4 h-4" />
            <span class="text-sm">Rotar NÚMEROS también</span>
          </label>
        </div>

        <div class="mt-4 flex gap-2">
          <button id="btnEncrypt" class="px-4 py-2 bg-blue-600 text-white rounded-md">Cifrar →</button>
          <button id="btnDecrypt" class="px-4 py-2 bg-green-600 text-white rounded-md">← Descifrar</button>
          <button id="btnBrute" class="px-4 py-2 bg-amber-500 text-slate-800 rounded-md">Probar todos</button>
          <button id="btnClear" class="px-4 py-2 bg-slate-200 rounded-md">Limpiar</button>
        </div>

        <div class="mt-3 flex gap-2">
          <button id="btnCopy" class="px-4 py-2 bg-slate-800 text-white rounded-md">Copiar resultado</button>
          <button id="btnSave" class="px-4 py-2 bg-slate-600 text-white rounded-md">Guardar .txt</button>
        </div>

      </div>

      <div>
        <label class="block text-sm font-medium text-slate-700">Resultado</label>
        <textarea id="outputText" rows="12" class="mt-2 p-3 w-full border rounded-md bg-slate-50" readonly></textarea>

        <div id="bruteBox" class="mt-3 hidden">
          <label class="block text-sm font-medium text-slate-700">Resultados (fuerza bruta)</label>
          <div id="bruteList" class="mt-2 space-y-2 max-h-48 overflow-auto p-2 border rounded-md bg-white"></div>
        </div>
      </div>
    </section>

    <footer class="mt-6 text-sm text-slate-500">Preserva mayúsculas. Los caracteres no definidos (puntuación, espacios) se mantienen.</footer>
  </main>

<script>
const ABC_LETRAS = "abcdefghijklmnopqrstuvwxyz";
const ABC_LETRAS_NUM = "abcdefghijklmnopqrstuvwxyz0123456789";

function shiftChar(ch, shift, dic) {
  const isUpper = ch === ch.toUpperCase() && ch.toLowerCase() !== ch.toUpperCase();
  const chLower = ch.toLowerCase();
  const idx = dic.indexOf(chLower);
  if (idx === -1) return ch;
  const newCh = dic[(idx + shift + dic.length) % dic.length];
  return isUpper ? newCh.toUpperCase() : newCh;
}

function transform(text, mov, rotNumbers, mode) {
  const dic = rotNumbers ? ABC_LETRAS_NUM : ABC_LETRAS;
  const shift = mode === 'enc' ? mov : -mov;
  let out = '';
  for (const ch of text) out += shiftChar(ch, shift, dic);
  return out;
}

// DOM
const inputText = document.getElementById('inputText');
const outputText = document.getElementById('outputText');
const shiftEl = document.getElementById('shift');
const rotNumbersEl = document.getElementById('rotNumbers');
const btnEncrypt = document.getElementById('btnEncrypt');
const btnDecrypt = document.getElementById('btnDecrypt');
const btnClear = document.getElementById('btnClear');
const btnCopy = document.getElementById('btnCopy');
const btnSave = document.getElementById('btnSave');
const btnBrute = document.getElementById('btnBrute');
const bruteBox = document.getElementById('bruteBox');
const bruteList = document.getElementById('bruteList');

btnEncrypt.addEventListener('click', () => {
  const text = inputText.value;
  const mov = parseInt(shiftEl.value) || 0;
  const rot = rotNumbersEl.checked;
  if (text.replace(/\s/g, '') !== '' && /^\d+$/.test(text.replace(/\s/g, '')) && !rot) {
    alert('Ha introducido solo números. Si desea cifrarlos active Rotar NÚMEROS.');
    return;
  }
  outputText.value = transform(text, mov, rot, 'enc');
  bruteBox.classList.add('hidden');
});

btnDecrypt.addEventListener('click', () => {
  const text = inputText.value;
  const mov = parseInt(shiftEl.value) || 0;
  const rot = rotNumbersEl.checked;
  if (text.replace(/\s/g, '') !== '' && /^\d+$/.test(text.replace(/\s/g, '')) && !rot) {
    alert('Imposible descifrar: introdujo solo números pero no se rotaron números.');
    return;
  }
  outputText.value = transform(text, mov, rot, 'dec');
  bruteBox.classList.add('hidden');
});

btnClear.addEventListener('click', () => {
  inputText.value = '';
  outputText.value = '';
  bruteBox.classList.add('hidden');
});

btnCopy.addEventListener('click', async () => {
  const t = outputText.value;
  if (!t) return alert('No hay resultado para copiar');
  try { await navigator.clipboard.writeText(t); alert('Copiado al portapapeles'); }
  catch (e) { alert('No se pudo copiar automáticamente. Seleccione y copie manualmente.'); }
});

btnSave.addEventListener('click', () => {
  const t = outputText.value;
  if (!t) return alert('No hay resultado para guardar');
  const blob = new Blob([t], {type: 'text/plain;charset=utf-8'});
  const url = URL.createObjectURL(blob);
  const a = document.createElement('a');
  a.href = url; a.download = 'resultado_cesar.txt';
  document.body.appendChild(a); a.click(); a.remove();
  URL.revokeObjectURL(url);
});

btnBrute.addEventListener('click', () => {
  const text = inputText.value;
  const rot = rotNumbersEl.checked;
  const dicLen = rot ? ABC_LETRAS_NUM.length : ABC_LETRAS.length;
  bruteList.innerHTML = '';
  for (let k=0;k<dicLen;k++){
    const out = transform(text, k, rot, 'dec');
    const item = document.createElement('div');
    item.className = 'p-2 border rounded-md bg-slate-50 flex justify-between items-start';
    item.innerHTML = `<div class="text-xs text-slate-500">k=${k}</div><pre class="whitespace-pre-wrap ml-3">${escapeHtml(out)}</pre>`;
    bruteList.appendChild(item);
  }
  bruteBox.classList.remove('hidden');
});

function escapeHtml(s){ return s.replace(/&/g,'&amp;').replace(/</g,'&lt;').replace(/>/g,'&gt;'); }
</script>
</body>
</html>
