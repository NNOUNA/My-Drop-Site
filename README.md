<!doctype html>
<html lang="fr">
<head>
<meta charset="utf-8">
<meta name="viewport" content="width=device-width,initial-scale=1">
<title>Calculateur surface de cap - Version corrigée</title>
<style>
body{font-family:system-ui,Arial;max-width:1150px;margin:20px auto;padding:16px}
label{font-weight:600;margin-top:6px;display:block}
input[type=number]{width:140px;padding:6px;margin-top:6px}
.zone{border:1px solid #ccc;padding:12px;border-radius:8px;margin:12px 0;background:#fafafa}
.result{background:#eef;padding:12px;border-radius:8px;margin-top:12px;font-size:15px}
img.schema{width:100%;max-width:750px;border:1px solid #ccc;border-radius:6px;margin:15px 0}
.btn{padding:8px 12px;margin:8px 6px}
.small{font-size:13px;color:#444}
</style>
</head>
<body>

<img src="https://gcertunisie.com/build/assets/gcer-logo-B9YKAQLs.svg" style="width:160px;margin-bottom:12px;">
<h1>Calculateur détaillé — Caps GRC (version corrigée)</h1>

<h2>Schéma explicatif des variables</h2>
<img class="schema" src="https://grand-maroon-t3y7mxbzgp.edgeone.app/tout%20variable%20caps%20(1).jpg" alt="Variables caps GRC" onerror="this.style.display='none'">

<h2>Variables globales</h2>
<label>D (diamètre moule) mm <input id="D" type="number" value="1000"></label>
<label>Épaisseur cap (Ep) mm <input id="Ep" type="number" value="10"></label>

<p><b>Paramètres calculés automatiquement depuis D :</b></p>
<div style="padding:10px;border:1px dashed #888;border-radius:6px;background:#f7f7f7;">
Ri = D (mm) → <span id="RiAff">?</span><br>
Rc = Ri × 0.1 (mm) → <span id="RcAff">?</span><br>
H1 calculé → <span id="H1Aff">?</span> mm<br>
H2 calculé → <span id="H2Aff">?</span> mm<br>
Somme H1+H2 → <span id="HsumAff">?</span> mm<br><br>
<b>Formule utilisée :</b><br>
H1 = tan((π/2 − β)/2) · sin(π/2 − β) · Ri<br>
H2 = sin(β) · Rc<br>
beta = <span id="BetaAff">63.612</span>°
</div>

<button id="autoFill" class="btn">Mettre à jour les paramètres liés à D</button>

<h2>Composition du cap</h2>

<div class="zone">
<label><input type="checkbox" id="zCalotte" checked> Partie calotte</label>
<div class="small">Formule : S = 2·π·(Ri+Ep)·(H1 + Ep·(1 − cos(26.3878°)))<br><small>H1 EXT après Ep = H1 + Ep·(1 − cos(26.3878°))</small></div>
</div>

<div class="zone">
<label><input type="checkbox" id="zTorique" checked> Partie torique</label>
<div class="small">Formule : S = (rc + ep) · RAD(alpha) · 2·π · ((D − 2·rc)/2) + ( (rc + ep) · SIN(RAD(alpha)) / RAD(alpha) )<br>
<small><b>Où :</b> alpha = 63.6122° &nbsp; | &nbsp; rc = Ri×0.1</small></div>
</div>

<div class="zone">
<label><input type="checkbox" id="zCyl" checked> Partie cylindrique</label>
<div class="small">Formule : S = π·(D+2Ep)·H3</div>
<label>Hauteur totale H mm <input id="H" type="number" value="200"></label>
<div>H3 = H − H1 − H2 → <span id="H3Aff">?</span> mm</div>
</div>

<div class="zone">
<label><input type="checkbox" id="zRenfort" checked> Partie renfort</label>
<div class="small">Formule : S = 100 · 2π · ( ((D + 2Ep)/2 + 63.66) − (63.66 * 2/π) ) // corrigée</div>
</div>

<div class="zone">
<label><input type="checkbox" id="zBride"> Partie bride</label>
<div id="brideInputs" style="display:none;margin-top:6px;">
<label>Épaisseur bride (EpBride) mm <input id="EpBride" type="number" value="10"></label>
<label>Diamètre extérieur bride (DextBride) mm <input id="DextBride" type="number" value="1200"></label>
Formule : S = π·(RextBride² – (Rint + Ep)²)
</div>
</div>

<button id="calc" class="btn">Calculer les surfaces</button>

<h2>Résultats</h2>
<div class="result" id="summary"></div>

<script>
// Utilitaires
function mm(v){return Number(v)||0;}

// Calculs automatiques basés sur D
function computeAuto(){
  const Dv = mm(document.getElementById('D').value);
  const Ri = Dv;              // selon ta règle Ri = D
  const Rc = Ri * 0.1;        // Rc = 0.1 * Ri
  const betaDeg = 63.612;     // beta en degrés (fixe)
  const beta = betaDeg * Math.PI/180;
  const H2 = Math.sin(beta) * Rc;
  const H1 = Math.tan((Math.PI/2 - beta)/2) * Math.sin(Math.PI/2 - beta) * Ri;
  return {Dv, Ri, Rc, beta, H1, H2};
}

// Mettre à jour l'affichage des paramètres calculés
function updateAutoDisplay(){
  const o = computeAuto();
  document.getElementById('RiAff').textContent = o.Ri.toFixed(3);
  document.getElementById('RcAff').textContent = o.Rc.toFixed(3);
  document.getElementById('H1Aff').textContent = o.H1.toFixed(3);
  document.getElementById('H2Aff').textContent = o.H2.toFixed(3);
  document.getElementById('HsumAff').textContent = (o.H1 + o.H2).toFixed(3);
  document.getElementById('BetaAff').textContent = '63.612';
  // update H3 and H display if H exists
  const Hval = mm(document.getElementById('H').value);
  document.getElementById('H3Aff').textContent = Math.max(0, Hval - o.H1 - o.H2).toFixed(3);
  document.getElementById('HAff').textContent = Hval;
}

// Evenements
document.getElementById('autoFill').addEventListener('click', ()=> updateAutoDisplay());
// update on D or Ep change also
document.getElementById('D').addEventListener('input', ()=> updateAutoDisplay());
document.getElementById('Ep').addEventListener('input', ()=> updateAutoDisplay());

document.getElementById('zBride').addEventListener('change', ()=>{
  document.getElementById('brideInputs').style.display = document.getElementById('zBride').checked ? 'block' : 'none';
});

// Calcul des surfaces
document.getElementById('calc').addEventListener('click', ()=>{
  const Epv = mm(document.getElementById('Ep').value);
  const Hv = mm(document.getElementById('H').value);
  const o = computeAuto();
  const pi = Math.PI;
  let total = 0;
  let out = '';

  // calotte
  if(document.getElementById('zCalotte').checked){
    const H1ext = o.H1 + Epv * (1 - Math.cos(26.3878 * Math.PI/180));
    const S = 2 * pi * (o.Ri + Epv) * H1ext;
    total += S; out += `Calotte = ${S.toFixed(3)} mm²<br>`;
  }

  // torique - formule Excel exacte sans Rctor
  if(document.getElementById('zTorique').checked){
    const alphaDeg = 63.6122;
    const Alpha = alphaDeg * Math.PI/180; // radians
    const rc = o.Rc; // rc = 0.1 * Ri
    const S = (rc + Epv) * Alpha * 2 * pi * (((o.Dv - 2*rc) / 2)
            + (((rc + Epv) * (Math.sin(Alpha) / Alpha))));
    total += S; out += `Torique = ${S.toFixed(3)} mm²<br>`;
  }

  // cylindrique
  if(document.getElementById('zCyl').checked){
    const H3 = Math.max(0, Hv - o.H1 - o.H2);
    const S = pi * (o.Dv + 2*Epv) * H3;
    total += S; out += `Cylindre = ${S.toFixed(3)} mm² (H3=${H3.toFixed(3)} mm)<br>`;
  }

  // renfort
  if(document.getElementById('zRenfort').checked){
    const S = 100 * 2 * pi * (((o.Dv + 2*Epv)/2 + 63.66) - (63.66 * 2/pi));
    total += S; out += `Renfort (approx) = ${S.toFixed(3)} mm²<br>`;
  }

  // bride
  if(document.getElementById('zBride').checked){
    const Rint = o.Dv/2;
    const RextB = mm(document.getElementById('DextBride').value)/2;
    const S = pi * (RextB*RextB - (Rint + Epv)*(Rint + Epv));
    total += S; out += `Bride = ${S.toFixed(3)} mm²<br>`;
  }

  document.getElementById('summary').innerHTML = out + `<hr><b>Total = ${total.toFixed(3)} mm²<br>Total = ${(total/1e6).toFixed(6)} m²</b>`;
});

// initial update
updateAutoDisplay();
</script>

</body>
</html>
