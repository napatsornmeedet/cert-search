<!DOCTYPE html>
<html lang="th">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1">
<title>ระบบค้นหาเกียรติบัตร</title>
<link rel="preconnect" href="https://fonts.googleapis.com">
<link rel="preconnect" href="https://fonts.gstatic.com" crossorigin>
<link href="https://fonts.googleapis.com/css2?family=Noto+Sans+Thai:wght@300;400;500;600;700&display=swap" rel="stylesheet">
<style>
  :root{
    --green:#006664;       /* เขียว ม.เกษตร */
    --green-dark:#004d4b;
    --green-soft:#e8f2f1;
    --yellow:#f2b705;
    --text:#1f2d2c;
    --muted:#5f6f6e;
    --line:#d6e3e2;
  }
  *{box-sizing:border-box}
  html,body{margin:0;min-height:100%}
  body{
    font-family:'Noto Sans Thai',sans-serif;
    color:var(--text);
    background:#fff;
    display:flex;flex-direction:column;min-height:100vh;
    padding:0 16px;
  }
  header{text-align:center;padding:36px 0 8px}
  header img{max-width:150px;max-height:150px;width:auto;height:auto;display:block;margin:0 auto 14px}
  header h1{margin:0;font-size:1.6rem;font-weight:700;color:var(--green)}
  header p{margin:6px 0 0;font-size:.95rem;color:var(--muted);line-height:1.6}
  .bar{width:80px;height:4px;background:var(--yellow);border-radius:4px;margin:16px auto 0}

  main{flex:1;width:100%;max-width:680px;margin:0 auto;padding:24px 0}
  .search{display:flex;gap:10px}
  .search input{
    flex:1;min-width:0;padding:14px 16px;font:inherit;font-size:1rem;
    border:2px solid var(--line);border-radius:10px;outline:none;transition:border-color .2s
  }
  .search input:focus{border-color:var(--green)}
  .btn{
    font:inherit;font-weight:600;cursor:pointer;border:0;border-radius:10px;
    padding:14px 24px;background:var(--green);color:#fff;transition:background .2s
  }
  .btn:hover{background:var(--green-dark)}
  .btn:disabled{opacity:.6;cursor:wait}
  .hint{margin:10px 2px 0;font-size:.85rem;color:var(--muted)}

  #result{margin-top:24px;display:grid;gap:16px}
  .card{
    border:1px solid var(--line);border-left:6px solid var(--green);
    border-radius:12px;padding:20px 22px;background:#fff;
    box-shadow:0 2px 10px rgba(0,102,100,.06)
  }
  .card .name{margin:0 0 14px;font-size:1.3rem;font-weight:700;color:var(--green);
    padding-bottom:10px;border-bottom:2px solid var(--yellow)}
  .row{margin:8px 0;line-height:1.6}
  .row .label{display:block;font-size:.8rem;color:var(--muted)}
  .row .value{font-weight:500}
  .dl{
    display:inline-block;margin-top:14px;padding:12px 26px;border-radius:10px;
    background:var(--green);color:#fff;text-decoration:none;font-weight:600;transition:background .2s
  }
  .dl:hover{background:var(--green-dark)}
  .dl .dot{color:var(--yellow)}
  .msg{
    text-align:center;padding:22px;border-radius:12px;background:var(--green-soft);
    color:var(--green-dark);font-weight:500
  }
  .msg.err{background:#fdf1f0;color:#a33}

  footer{text-align:center;padding:24px 0 28px;font-size:.88rem;color:var(--muted);line-height:1.6}
</style>
</head>
<body>

<header>
  <!-- ใส่โลโก้ของท่านที่ LOGO_URL ด้านล่าง (ขนาดไม่เกิน 150px) -->
  <img id="logo" alt="โลโก้" style="display:none">
  <h1>ระบบค้นหาเกียรติบัตร</h1>
  <p>สถาบันวิจัยและพัฒนาแห่งมหาวิทยาลัยเกษตรศาสตร์<br>ฝ่ายมาตรฐานการวิจัยและศูนย์สัตว์ทดลอง</p>
  <div class="bar"></div>
</header>

<main>
  <div class="search">
    <input id="q" type="text" placeholder="กรอกชื่อ หรือ นามสกุล" autocomplete="off">
    <button id="go" class="btn">ค้นหา</button>
  </div>
  <p class="hint">พิมพ์ชื่อหรือนามสกุลอย่างน้อย 2 ตัวอักษร แล้วกด Enter หรือปุ่มค้นหา</p>
  <div id="result"></div>
</main>

<footer>สถาบันวิจัยและพัฒนาแห่งมหาวิทยาลัยเกษตรศาสตร์ ฝ่ายมาตรฐานการวิจัยและศูนย์สัตว์ทดลอง</footer>

<script>
  // ===== ตั้งค่า =====
  const APPS_SCRIPT_URL = "https://script.google.com/macros/s/AKfycbyJtLtBVWQ5ufoixuS5Ym1OOTswox6Pq1v5NYxHSAFWo1BwYefa5u8tahzQDLmjffOI/exec";   // URL ของ Web App (ลงท้ายด้วย /exec)
  const LOGO_URL = "";              // ใส่ลิงก์/พาธโลโก้ เช่น "logo.png"
  // ====================

  const $ = id => document.getElementById(id);
  const input = $("q"), btn = $("go"), out = $("result");

  if (LOGO_URL) { const l = $("logo"); l.src = LOGO_URL; l.style.display = "block"; }

  function el(tag, cls, text){
    const e = document.createElement(tag);
    if (cls) e.className = cls;
    if (text !== undefined) e.textContent = text;
    return e;
  }
  function row(label, value){
    const r = el("div","row");
    r.append(el("span","label",label), el("span","value",value));
    return r;
  }
  function showMsg(text, isErr){
    out.replaceChildren(el("div","msg"+(isErr?" err":""), text));
  }

  function render(items){
    out.replaceChildren();
    items.forEach(it => {
      const card = el("div","card");
      card.append(
        el("h2","name", [it.prename, it.firstname, it.lastname].filter(Boolean).join(" ").replace(/\s+/g," ")),
        row("รหัสเกียรติบัตร", it.id),
        row("ชื่อการอบรม", it.title)
      );
      if (it.pdfUrl){
        const a = el("a","dl");
        a.href = it.pdfUrl; a.target = "_blank"; a.rel = "noopener noreferrer";
        a.append(el("span","dot","● "), document.createTextNode("ดาวน์โหลด PDF"));
        card.append(a);
      }
      out.append(card);
    });
  }

  async function search(){
    const q = input.value.trim();
    if (q.length < 2){ showMsg("กรุณากรอกชื่อหรือนามสกุลอย่างน้อย 2 ตัวอักษร", true); return; }
    btn.disabled = true; btn.textContent = "กำลังค้นหา...";
    showMsg("กำลังค้นหาข้อมูล...");
    try{
      const res = await fetch(APPS_SCRIPT_URL + "?q=" + encodeURIComponent(q));
      const data = await res.json();
      if (!data.ok) throw new Error(data.error || "error");
      if (!data.results.length) showMsg("ไม่พบข้อมูลของคุณในระบบ");
      else render(data.results);
    }catch(err){
      showMsg("เกิดข้อผิดพลาดในการเชื่อมต่อ กรุณาลองใหม่อีกครั้ง", true);
    }finally{
      btn.disabled = false; btn.textContent = "ค้นหา";
    }
  }

  btn.addEventListener("click", search);
  input.addEventListener("keydown", e => { if (e.key === "Enter") search(); });
</script>
</body>
</html>
