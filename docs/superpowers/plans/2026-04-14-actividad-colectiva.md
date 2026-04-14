# Actividad Colectiva — Implementation Plan

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** Add "Actividad colectiva" as a new quota type — with collective minimum goal, per-student opt-in commitments, individual payment tracking, and semi-automatic gap coverage from course funds.

**Architecture:** Single `index.html` file (~4000 lines) with vanilla JS. Actividades are stored in `state.quotas` with `tipo: "actividad"` discriminator. Core helpers `isInQuota`, `getStudentAmount`, `getQuotaStudents` are updated to handle the new type. All rendering follows existing sheet/card patterns.

**Tech Stack:** Vanilla HTML/JS, Firebase Realtime Database, no build step.

---

## File Map

Only file modified: `index.html`

Key line anchors (verify before editing — file grows with each commit):
- `isInQuota` → ~1179
- `getStudentAmount` → ~1183
- `getQuotaStudents` → ~1195
- `renderCuotas` → ~1305
- `APP_VERSION` → ~1673
- `openAddQuota` → ~1767
- `saveAddQuota` → ~1976
- `deleteQuota` (one-liner) → ~1541

---

## Task 1: Type selector in openAddQuota

**File:** `index.html` — modify `openAddQuota()` (~line 1767)

**What:** Add a two-button type selector (Cuota / Actividad colectiva) at the top of the sheet. Wrap the existing form in `<div id="q-cuota-form">`. Add a new hidden `<div id="q-actividad-form">` below it.

- [ ] **Step 1.1: Locate the `qBodyHtml` construction in `openAddQuota`**

  Find the block starting at ~line 1841:
  ```js
  var qBodyHtml = field("Nombre","f-name","ej: Asado fin de año")
    +dateField("Fecha","f-date",hoy())
    +draftToggle
    + '<div id="q-monto-wrap">'+moneyInput("Monto por alumno ($)","f-amount","ej: 5.000")+'</div>'
    +allStudentsHTML;
  ```

- [ ] **Step 1.2: Replace that block with the new version that includes the type selector**

  ```js
  var typeSelector = '<div style="display:flex;gap:8px;margin-bottom:16px">'
    + '<button type="button" id="qtype-cuota" onclick="qSetType(\'cuota\')" style="flex:1;padding:10px;border-radius:8px;border:1px solid var(--navy);background:var(--navy);color:#fff;font-size:13px;font-weight:600;cursor:pointer">📋 Cuota</button>'
    + '<button type="button" id="qtype-actividad" onclick="qSetType(\'actividad\')" style="flex:1;padding:10px;border-radius:8px;border:1px solid var(--border2);background:transparent;color:var(--muted);font-size:13px;cursor:pointer">🎯 Actividad colectiva</button>'
    + '<input type="hidden" id="q-tipo-val" value="cuota">'
    + '</div>';

  var actividadForm = '<div id="q-actividad-form" style="display:none">'
    + field("Nombre","act-name","ej: Bingo Solidario")
    + moneyInput("Precio por unidad ($)","act-precio","ej: 10.000")
    + '<div class="field"><label>Meta mínima (unidades)</label><input id="act-meta" type="number" min="1" placeholder="ej: 15" style="width:100%;padding:13px 14px;background:var(--bg);border:1.5px solid var(--border2);border-radius:var(--r);color:var(--text);font-size:15px;outline:none;font-family:Roboto,sans-serif"></div>'
    + dateField("Fecha","act-date",hoy())
    + '</div>';

  var qBodyHtml = typeSelector
    + '<div id="q-cuota-form">'
    + field("Nombre","f-name","ej: Asado fin de año")
    + dateField("Fecha","f-date",hoy())
    + draftToggle
    + '<div id="q-monto-wrap">'+moneyInput("Monto por alumno ($)","f-amount","ej: 5.000")+'</div>'
    + allStudentsHTML
    + '</div>'
    + actividadForm;
  ```

- [ ] **Step 1.3: Update the save button listener (line ~1847) to dispatch by type**

  Current:
  ```js
  document.getElementById("quota-save-btn").addEventListener("click", saveAddQuota);
  ```
  Replace with:
  ```js
  document.getElementById("quota-save-btn").addEventListener("click", function(){
    if(document.getElementById("q-tipo-val").value === "actividad"){
      saveAddActividad();
    } else {
      saveAddQuota();
    }
  });
  ```

- [ ] **Step 1.4: Manual test — open "Nueva cuota" and verify two buttons appear, clicking each toggles the form (actividad form not visible yet, it just needs to not crash)**

---

## Task 2: qSetType + saveAddActividad functions

**File:** `index.html` — add two new functions after `saveAddQuota` (~line 2029)

- [ ] **Step 2.1: Add `qSetType` function immediately after `openAddQuota` closes (~line 1848)**

  ```js
  function qSetType(tipo){
    document.getElementById("q-tipo-val").value=tipo;
    var isCuota=tipo==="cuota";
    document.getElementById("q-cuota-form").style.display=isCuota?"block":"none";
    document.getElementById("q-actividad-form").style.display=isCuota?"none":"block";
    var btnC=document.getElementById("qtype-cuota");
    var btnA=document.getElementById("qtype-actividad");
    if(btnC){btnC.style.background=isCuota?"var(--navy)":"transparent";btnC.style.color=isCuota?"#fff":"var(--muted)";btnC.style.borderColor=isCuota?"var(--navy)":"var(--border2)";}
    if(btnA){btnA.style.background=isCuota?"transparent":"var(--navy)";btnA.style.color=isCuota?"var(--muted)":"#fff";btnA.style.borderColor=isCuota?"var(--border2)":"var(--navy)";}
  }
  ```

- [ ] **Step 2.2: Add `saveAddActividad` function after `saveAddQuota` (~line 2029)**

  ```js
  function saveAddActividad(){
    var name=document.getElementById("act-name").value.trim();
    var precioUnidad=getMoneyVal("act-precio");
    var metaMinima=parseInt(document.getElementById("act-meta").value)||0;
    var date=document.getElementById("act-date").value;
    if(!name||!precioUnidad||!metaMinima){showToast("Completa todos los campos","err");return;}
    state.quotas.push({
      id:uid(),
      tipo:"actividad",
      name:name,
      precioUnidad:precioUnidad,
      metaMinima:metaMinima,
      estado:"abierta",
      compromisos:{},
      gastoId:null,
      date:date,
      amount:0,
      studentIds:[],
      amounts:{},
      draft:false
    });
    addLog("🎯","Actividad creada: \""+name+"\" — "+fmt(precioUnidad)+"/u, meta "+metaMinima+" unidades");
    saveData();closeSheet();render();showToast("✓ Actividad creada");
  }
  ```

- [ ] **Step 2.3: Manual test — click "🎯 Actividad colectiva", fill name + precio + meta + fecha, save. Open Firebase console and verify the new object has `tipo:"actividad"` and `compromisos:{}`**

---

## Task 3: Update isInQuota, getStudentAmount, getQuotaStudents

**File:** `index.html` — modify three functions at ~lines 1179–1207

These changes make the existing payment/stats system work correctly for actividades without any other changes to `renderPagos`, `qStats`, `isPaid`, or `renderPendientes`.

- [ ] **Step 3.1: Replace `isInQuota` (line ~1179)**

  Current:
  ```js
  function isInQuota(sid,q){
    if(!q.studentIds||q.studentIds.length===0)return true;
    return q.studentIds.includes(sid);
  }
  ```
  Replace with:
  ```js
  function isInQuota(sid,q){
    if(q.tipo==="actividad"){
      return q.compromisos&&(q.compromisos[sid]||0)>0;
    }
    if(!q.studentIds||q.studentIds.length===0)return true;
    return q.studentIds.includes(sid);
  }
  ```

- [ ] **Step 3.2: Replace `getStudentAmount` (line ~1183)**

  Current:
  ```js
  function getStudentAmount(sid,q){
    if(q.amounts&&q.amounts[sid]!==undefined)return q.amounts[sid];
    return q.amount;
  }
  ```
  Replace with:
  ```js
  function getStudentAmount(sid,q){
    if(q.tipo==="actividad"){
      return ((q.compromisos&&q.compromisos[sid])||0)*(q.precioUnidad||0);
    }
    if(q.amounts&&q.amounts[sid]!==undefined)return q.amounts[sid];
    return q.amount;
  }
  ```

- [ ] **Step 3.3: Replace `getQuotaStudents` (line ~1195)**

  Current:
  ```js
  function getQuotaStudents(q){
    if(q.studentIds&&q.studentIds.length>0) return state.students.filter(s=>q.studentIds.includes(s.id));
    return state.students;
  }
  ```
  Replace with:
  ```js
  function getQuotaStudents(q){
    if(q.tipo==="actividad"){
      var comp=q.compromisos||{};
      return state.students.filter(function(s){return(comp[s.id]||0)>0;});
    }
    if(q.studentIds&&q.studentIds.length>0) return state.students.filter(s=>q.studentIds.includes(s.id));
    return state.students;
  }
  ```

- [ ] **Step 3.4: No test needed here — these functions are pure logic. They'll be exercised in Task 5.**

---

## Task 4: Update renderCuotas — actividades section

**File:** `index.html` — modify `renderCuotas` (~line 1305)

- [ ] **Step 4.1: Change the `activeQ` filter to exclude actividades and add actividades array**

  Current first two lines of `renderCuotas`:
  ```js
  const activeQ=state.quotas.filter(q=>!q.draft);
  const drafts=state.quotas.filter(q=>q.draft);
  ```
  Replace with:
  ```js
  const activeQ=state.quotas.filter(q=>!q.draft&&q.tipo!=="actividad");
  const drafts=state.quotas.filter(q=>q.draft&&q.tipo!=="actividad");
  const actividades=state.quotas.filter(q=>q.tipo==="actividad");
  ```

- [ ] **Step 4.2: Add actividades section at the end of `renderCuotas`, just before the final `return html`**

  Find the line `if(!isDesktop()&&isAdmin) html+=...` near the end of the function (line ~1355). Add after `html+=\`</div>\`` (closing the activeQ section):

  ```js
  if(actividades.length){
    html+='<div class="section"><div class="sec-title">🎯 Actividades colectivas ('+actividades.length+')</div>';
    actividades.forEach(function(q){
      var comp=q.compromisos||{};
      var totalU=Object.values(comp).reduce(function(s,v){return s+v;},0);
      var totalM=totalU*(q.precioUnidad||0);
      var pct=q.metaMinima>0?Math.min(100,Math.round(totalU/q.metaMinima*100)):0;
      var gap=Math.max(0,q.metaMinima-totalU);
      var estadoBadge=q.estado==="cerrada"
        ?'<span style="font-size:10px;padding:2px 7px;border-radius:10px;background:var(--surface2);color:var(--muted)">Cerrada</span>'
        :'<span style="font-size:10px;padding:2px 7px;border-radius:10px;background:var(--green-dim);color:var(--green)">Abierta</span>';
      html+='<div class="card" style="margin-bottom:10px">'
        +'<div class="card-top">'
        +'<div style="flex:1;min-width:0">'
        +'<div class="card-title">'+q.name+'</div>'
        +'<div class="card-meta">'+fmt(q.precioUnidad)+'/u · Meta: '+q.metaMinima+' · '+q.date+'</div>'
        +'</div>'
        +'<div style="display:flex;align-items:center;gap:6px">'
        +estadoBadge
        +(isAdmin?'<button class="icon-btn" onclick="requireAdmin(()=>openActividadPanel(\''+q.id+'\'))">⚙️</button>':'')
        +'<button class="icon-btn" onclick="openActividadPanel(\''+q.id+'\')" title="Ver">👁</button>'
        +(isAdmin?'<button class="icon-btn" onclick="requireAdmin(()=>deleteQuota(\''+q.id+'\'))">🗑️</button>':'')
        +'</div>'
        +'</div>'
        +'<div class="prog-bar"><div class="prog-fill" style="width:'+pct+'%;background:'+(pct>=100?'#3dd68c':'#2d5be3')+'"></div></div>'
        +'<div class="card-footer">'
        +'<span style="color:#3dd68c">✓ '+totalU+' comprometidas</span>'
        +'<span style="color:'+(gap>0?'var(--yellow)':'#3dd68c')+'">'+( gap>0?'⚠ Faltan '+gap:'✓ Meta cubierta')+'</span>'
        +'<span style="color:#7a85a0">'+fmt(totalM)+' esperado</span>'
        +'</div>'
        +'</div>';
    });
    html+='</div>';
  }
  ```

- [ ] **Step 4.3: Manual test — create an actividad via Task 2 and verify it appears in a new "Actividades colectivas" section in the Cuotas tab, separate from regular cuotas**

---

## Task 5: openActividadPanel + actAdjUnit + actUpdateHeader

**File:** `index.html` — add three new functions after `saveAddActividad`

- [ ] **Step 5.1: Add `openActividadPanel(id)` function**

  ```js
  function openActividadPanel(id){
    var q=state.quotas.find(function(x){return x.id===id;});
    if(!q)return;
    var comp=q.compromisos||{};
    var totalU=Object.values(comp).reduce(function(s,v){return s+v;},0);
    var totalM=totalU*(q.precioUnidad||0);
    var pct=q.metaMinima>0?Math.min(100,Math.round(totalU/q.metaMinima*100)):0;
    var gap=Math.max(0,q.metaMinima-totalU);

    var header='<div style="background:var(--surface2);border-radius:var(--r);padding:14px;margin-bottom:16px">'
      +'<div style="font-weight:700;font-size:15px;margin-bottom:4px">'+q.name+'</div>'
      +'<div style="font-size:12px;color:var(--muted);margin-bottom:10px">'+fmt(q.precioUnidad)+' por unidad · Meta: '+q.metaMinima+' unidades</div>'
      +'<div style="display:flex;justify-content:space-between;font-size:12px;margin-bottom:6px">'
      +'<span id="act-prog-label" style="color:var(--navy-light);font-weight:600">'+totalU+' / '+q.metaMinima+' unidades</span>'
      +'<span id="act-prog-monto" style="color:var(--green);font-weight:600">'+fmt(totalM)+'</span>'
      +'</div>'
      +'<div style="background:var(--border2);border-radius:4px;height:6px;overflow:hidden">'
      +'<div id="act-prog-fill" style="width:'+pct+'%;height:100%;background:'+(pct>=100?'var(--green)':'var(--navy)')+';border-radius:4px;transition:width .3s"></div>'
      +'</div>'
      +'<div id="act-gap-label" style="font-size:11px;margin-top:6px;color:'+(gap>0?'var(--yellow)':'var(--green)')+'">'+( gap>0?'⚠ Faltan '+gap+' unidades para la meta':'✓ Meta alcanzada')+'</div>'
      +'</div>';

    var listHeader='<div style="display:flex;align-items:center;padding:6px 2px;border-bottom:2px solid var(--border2);margin-bottom:4px">'
      +'<span style="flex:1;font-size:11px;color:var(--muted);font-weight:600">Familia</span>'
      +'<span style="width:90px;text-align:center;font-size:11px;color:var(--muted);font-weight:600">Unidades</span>'
      +'<span style="width:70px;text-align:right;font-size:11px;color:var(--muted);font-weight:600">Monto</span>'
      +'<span style="width:40px;text-align:right;font-size:11px;color:var(--muted);font-weight:600">Pago</span>'
      +'</div>';

    var rows=state.students.filter(function(s){return!s.paused;}).map(function(s){
      var units=comp[s.id]||0;
      var monto=units*(q.precioUnidad||0);
      var pagado=isPaid(s.id,q.id);
      var ap=s.name.split(",")[0]||s.name;
      return '<div style="display:flex;align-items:center;gap:4px;padding:8px 2px;border-bottom:1px solid var(--border)">'
        +'<span style="flex:1;font-size:13px">'+ap+'</span>'
        +'<div style="width:90px;display:flex;align-items:center;gap:2px;justify-content:center">'
        +'<button type="button" onclick="actAdjUnit(\''+id+'\',\''+s.id+'\',-1)" style="width:24px;height:24px;border-radius:4px;border:1px solid var(--border2);background:var(--surface2);color:var(--text);font-size:14px;cursor:pointer;font-family:Roboto,sans-serif">−</button>'
        +'<span id="actunit-'+s.id+'" style="font-size:13px;font-weight:600;min-width:22px;text-align:center">'+units+'</span>'
        +'<button type="button" onclick="actAdjUnit(\''+id+'\',\''+s.id+'\',1)" style="width:24px;height:24px;border-radius:4px;border:1px solid var(--border2);background:var(--surface2);color:var(--text);font-size:14px;cursor:pointer;font-family:Roboto,sans-serif">+</button>'
        +'</div>'
        +'<span id="actmonto-'+s.id+'" style="width:70px;text-align:right;font-size:12px;color:var(--muted)">'+(monto>0?fmt(monto):'—')+'</span>'
        +'<span style="width:40px;text-align:right;font-size:12px;color:'+(pagado?'var(--green)':(units>0?'var(--yellow)':'var(--muted)'))+'">'+(units>0?(pagado?'✓':'⏳'):'—')+'</span>'
        +'</div>';
    }).join('');

    var isCerrada=q.estado==="cerrada";
    var footerHtml=isCerrada
      ?'<div style="text-align:center;color:var(--muted);font-size:13px;padding:4px 0">Actividad cerrada'+(q.gastoId?'<span style="color:var(--green)"> · Gasto registrado ✓</span>':'')+'</div>'
      :'<div style="display:flex;gap:8px">'
      +'<button class="btn-cancel" onclick="closeSheet()">Cancelar</button>'
      +'<button class="btn-save" onclick="saveActividadCompromisos(\''+id+'\')">Guardar</button>'
      +(isAdmin?'<button onclick="requireAdmin(function(){cerrarActividad(\''+id+'\')})" style="padding:12px 16px;border-radius:var(--r);background:transparent;border:1px solid var(--border2);color:var(--muted);font-size:13px;cursor:pointer;font-family:Roboto,sans-serif">Cerrar</button>':'')
      +'</div>';

    document.getElementById("modal-container").innerHTML='<div class="overlay" id="overlay" onclick="maybeClose(event)">'
      +'<div class="sheet">'
      +'<div class="sheet-handle"><div class="sheet-bar"></div></div>'
      +'<div class="sheet-header"><span class="sheet-title">🎯 '+q.name+'</span><button class="sheet-close" onclick="closeSheet()">✕</button></div>'
      +'<div class="sheet-body">'+header+listHeader+'<div style="max-height:320px;overflow-y:auto">'+rows+'</div></div>'
      +'<div class="sheet-footer">'+footerHtml+'</div>'
      +'</div></div>';

    window._actCompromisos=Object.assign({},comp);
    window._actPrecioUnidad=q.precioUnidad||0;
    window._actMetaMinima=q.metaMinima||0;
    window._actId=id;
  }
  ```

- [ ] **Step 5.2: Add `actAdjUnit` function**

  ```js
  function actAdjUnit(actId,sid,delta){
    if(!window._actCompromisos)return;
    var current=window._actCompromisos[sid]||0;
    var next=Math.max(0,current+delta);
    window._actCompromisos[sid]=next;
    var unitEl=document.getElementById("actunit-"+sid);
    var montoEl=document.getElementById("actmonto-"+sid);
    if(unitEl)unitEl.textContent=next;
    if(montoEl)montoEl.textContent=next>0?fmt(next*window._actPrecioUnidad):"—";
    actUpdateHeader();
  }
  ```

- [ ] **Step 5.3: Add `actUpdateHeader` function**

  ```js
  function actUpdateHeader(){
    var totalU=Object.values(window._actCompromisos).reduce(function(s,v){return s+v;},0);
    var totalM=totalU*window._actPrecioUnidad;
    var pct=window._actMetaMinima>0?Math.min(100,Math.round(totalU/window._actMetaMinima*100)):0;
    var gap=Math.max(0,window._actMetaMinima-totalU);
    var fill=document.getElementById("act-prog-fill");
    var label=document.getElementById("act-prog-label");
    var monto=document.getElementById("act-prog-monto");
    var gapLbl=document.getElementById("act-gap-label");
    if(fill){fill.style.width=pct+"%";fill.style.background=pct>=100?"var(--green)":"var(--navy)";}
    if(label)label.textContent=totalU+" / "+window._actMetaMinima+" unidades";
    if(monto)monto.textContent=fmt(totalM);
    if(gapLbl){gapLbl.textContent=gap>0?"⚠ Faltan "+gap+" unidades para la meta":"✓ Meta alcanzada";gapLbl.style.color=gap>0?"var(--yellow)":"var(--green)";}
  }
  ```

- [ ] **Step 5.4: Manual test — click the ⚙️ button on an actividad. Verify panel opens with student list. Click +/− on a student and verify: unit count changes, monto updates, progress bar and label update live**

---

## Task 6: saveActividadCompromisos + cerrarActividad

**File:** `index.html` — add two new functions after `actUpdateHeader`

- [ ] **Step 6.1: Add `saveActividadCompromisos` function**

  ```js
  function saveActividadCompromisos(id){
    var q=state.quotas.find(function(x){return x.id===id;});
    if(!q)return;
    var comp={};
    Object.keys(window._actCompromisos||{}).forEach(function(sid){
      if(window._actCompromisos[sid]>0)comp[sid]=window._actCompromisos[sid];
    });
    state.quotas=state.quotas.map(function(x){
      return x.id===id?Object.assign({},x,{compromisos:comp}):x;
    });
    addLog("🎯","Compromisos actualizados: \""+q.name+"\"");
    saveData();closeSheet();render();showToast("✓ Cambios guardados");
  }
  ```

- [ ] **Step 6.2: Add `cerrarActividad` function**

  ```js
  function cerrarActividad(id){
    var q=state.quotas.find(function(x){return x.id===id;});
    if(!q)return;
    var comp=window._actCompromisos||q.compromisos||{};
    var totalU=Object.values(comp).reduce(function(s,v){return s+v;},0);
    var gap=Math.max(0,q.metaMinima-totalU);
    var gapMonto=gap*(q.precioUnidad||0);
    var cleanComp={};
    Object.keys(comp).forEach(function(sid){if(comp[sid]>0)cleanComp[sid]=comp[sid];});
    var gastoId=null;
    if(gap>0){
      var msg="Faltan "+gap+" unidades para la meta.\nEl curso cubriría "+fmt(gapMonto)+" del saldo.\n\n¿Registrar como gasto?";
      if(confirm(msg)){
        gastoId=uid();
        state.expenses.push({
          id:gastoId,
          name:"Actividad: "+q.name+" — aporte curso",
          amount:gapMonto,
          category:"Actividad",
          date:hoy(),
          note:gap+" unidades × "+fmt(q.precioUnidad)
        });
        addLog("💸","Gasto por gap actividad: \""+q.name+"\" — "+fmt(gapMonto));
      }
    }
    state.quotas=state.quotas.map(function(x){
      return x.id===id?Object.assign({},x,{compromisos:cleanComp,estado:"cerrada",gastoId:gastoId}):x;
    });
    addLog("🔒","Actividad cerrada: \""+q.name+"\"");
    saveData();closeSheet();render();showToast("✓ Actividad cerrada");
  }
  ```

- [ ] **Step 6.3: Manual test — gap scenario: create actividad meta=3, commit only 1 student (1 unidad), click Cerrar. Verify confirm dialog shows "Faltan 2 unidades" with correct monto. Confirm → verify gasto appears in tab Gastos with nombre "Actividad: [nombre] — aporte curso"**

- [ ] **Step 6.4: Manual test — no-gap scenario: create actividad meta=2, commit 2+ unidades total, click Cerrar. Verify no dialog appears and actividad shows badge "Cerrada"**

- [ ] **Step 6.5: Manual test — pagos tab: select the actividad from the pills. Verify only committed students appear in the list, with correct amounts (unidades × precioUnidad)**

---

## Task 7: Version bump + commit

**File:** `index.html` — line ~1673

- [ ] **Step 7.1: Update APP_VERSION**

  ```js
  const APP_VERSION="v2.10";
  ```

- [ ] **Step 7.2: Commit**

  ```bash
  git add index.html
  git commit -m "feat v2.10: actividad colectiva — tipo de cuota con meta mínima, compromisos opt-in y gap semi-automático"
  ```

- [ ] **Step 7.3: Deploy**

  ```bash
  git push origin v2
  vercel --prod --yes
  ```

---

## Self-Review Notes

- **Spec coverage:** Type selector ✓ · Wizard con 3 campos ✓ · Panel gestión con barra progreso ✓ · Lista editable en tiempo real ✓ · Cierre con gap semi-automático ✓ · Pagos integrados via isInQuota/getStudentAmount ✓ · estado abierta/cerrada ✓
- **No placeholders:** All steps have actual code
- **Type consistency:** `compromisos`, `precioUnidad`, `metaMinima`, `estado`, `gastoId` used identically across all tasks
- **Backward compat:** `mergeState` at ~line 968 maps `quotas` with spread — new fields survive Firebase round-trip. Regular quotas unaffected (tipo undefined ≠ "actividad")
