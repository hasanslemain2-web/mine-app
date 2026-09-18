from pathlib import Path
import zipfile, shutil, json, textwrap

root = Path("/mnt/data/MINE_PWA")
if root.exists():
    shutil.rmtree(root)
root.mkdir(parents=True)

index = r'''<!doctype html>
<html lang="ar" dir="rtl">
<head>
<meta charset="utf-8"><meta name="viewport" content="width=device-width,initial-scale=1,viewport-fit=cover">
<meta name="theme-color" content="#10131a"><link rel="manifest" href="manifest.json">
<title>MINE — نسخة مني</title>
<style>
*{box-sizing:border-box}body{margin:0;background:#0b0e13;color:#f5f7fb;font-family:system-ui,-apple-system,Segoe UI,sans-serif}
.app{max-width:520px;margin:auto;min-height:100vh;background:#11151d}
header{padding:22px 20px 12px;position:sticky;top:0;background:#11151dee;backdrop-filter:blur(12px);z-index:5}
.brand{font-size:25px;font-weight:800}.sub{color:#8f98a8;font-size:13px;margin-top:3px}
main{padding:14px 16px 95px}.card{background:#191f29;border:1px solid #252d39;border-radius:20px;padding:18px;margin:12px 0}
h2{margin:0 0 8px;font-size:20px}h3{margin:0 0 6px}.muted{color:#9da6b5;line-height:1.6}
.row{display:flex;gap:10px;align-items:center}.grow{flex:1}
.btn{border:0;border-radius:14px;padding:12px 15px;background:#6d5dfc;color:#fff;font-weight:700}
.btn.secondary{background:#252d39}.btn.danger{background:#43242b}.input{width:100%;padding:14px;border-radius:14px;border:1px solid #303947;background:#0f131a;color:#fff;font-size:15px}
.chip{display:inline-block;padding:8px 11px;background:#222a36;border-radius:999px;margin:4px;font-size:13px}
.nav{position:fixed;bottom:0;left:0;right:0;max-width:520px;margin:auto;background:#11151df2;border-top:1px solid #28303c;display:grid;grid-template-columns:repeat(5,1fr);padding:8px 7px calc(8px + env(safe-area-inset-bottom));z-index:10}
.nav button{background:none;border:0;color:#7f8999;font-size:11px;padding:7px 2px}.nav button.active{color:#fff}.icon{font-size:20px;display:block;margin-bottom:3px}
.msg{padding:12px 14px;border-radius:17px;margin:9px 0;max-width:88%;line-height:1.55}.me{background:#6d5dfc;margin-right:auto}.ai{background:#202733;margin-left:auto}
.check{display:flex;gap:10px;align-items:center;padding:12px 0;border-bottom:1px solid #2a323e}.check:last-child{border:0}
</style></head>
<body><div class="app">
<header><div class="brand">MINE — نسخة مني</div><div class="sub">مساعدك الشخصي، تحت سيطرتك</div></header>
<main id="main"></main>
<nav class="nav">
<button onclick="go('home')" id="n-home"><span class="icon">⌂</span>الرئيسية</button>
<button onclick="go('chat')" id="n-chat"><span class="icon">◌</span>المحادثة</button>
<button onclick="go('memory')" id="n-memory"><span class="icon">◈</span>الذاكرة</button>
<button onclick="go('goals')" id="n-goals"><span class="icon">✓</span>الأهداف</button>
<button onclick="go('decide')" id="n-decide"><span class="icon">?</span>القرار</button>
</nav></div>
<script>
const S={mem:JSON.parse(localStorage.getItem('mine_mem')||'[]'),goals:JSON.parse(localStorage.getItem('mine_goals')||'[]'),chat:[]};
function save(){localStorage.setItem('mine_mem',JSON.stringify(S.mem));localStorage.setItem('mine_goals',JSON.stringify(S.goals))}
function go(p){document.querySelectorAll('.nav button').forEach(x=>x.classList.remove('active'));let n=document.getElementById('n-'+p);if(n)n.classList.add('active');
let h='';
if(p==='home')h=`<div class="card"><h2>مرحباً 👋</h2><p class="muted">MINE يتعلم تفضيلاتك ليساعدك على التفكير واتخاذ قرارات أوضح.</p><div class="row"><button class="btn" onclick="go('chat')">ابدأ المحادثة</button><button class="btn secondary" onclick="go('memory')">ذاكرتي</button></div></div>
<div class="card"><h3>لمحة سريعة</h3><p class="muted">ذاكرة: ${S.mem.length} • أهداف: ${S.goals.length}</p></div>
<div class="card"><h3>مبدأ MINE</h3><p class="muted">الذكاء الاصطناعي لا يدّعي أنه أنت. يعرض المعلومات والأسباب، وأنت صاحب القرار.</p></div>`;
if(p==='chat')h=`<div class="card"><h2>المحادثة</h2><div id="chatbox">${S.chat.map(x=>`<div class="msg ${x.who}">${x.t}</div>`).join('')}</div><div class="row"><input id="chatin" class="input" placeholder="اكتب رسالتك..."><button class="btn" onclick="send()">إرسال</button></div></div>`;
if(p==='memory')h=`<div class="card"><h2>ذاكرتي</h2><p class="muted">أضف تفضيلات تريد أن يستخدمها التطبيق.</p><div class="row"><input id="memin" class="input" placeholder="مثلاً: أحب الاختصار"><button class="btn" onclick="addMem()">+</button></div>${S.mem.map((x,i)=>`<div class="check"><span class="grow">${x}</span><button class="btn danger" onclick="delMem(${i})">حذف</button></div>`).join('')}</div>`;
if(p==='goals')h=`<div class="card"><h2>الأهداف</h2><div class="row"><input id="goin" class="input" placeholder="هدف جديد"><button class="btn" onclick="addGoal()">+</button></div>${S.goals.map((x,i)=>`<div class="check"><input type="checkbox" onchange="toggleGoal(${i})" ${x.done?'checked':''}><span class="grow" style="${x.done?'text-decoration:line-through;color:#788190':''}">${x.t}</span><button class="btn danger" onclick="delGoal(${i})">حذف</button></div>`).join('')}</div>`;
if(p==='decide')h=`<div class="card"><h2>مساعد القرار</h2><p class="muted">اكتب خيارين وسأرتبهما لك حسب معاييرك، مع إبقاء القرار لك.</p><input id="a" class="input" placeholder="الخيار الأول"><br><br><input id="b" class="input" placeholder="الخيار الثاني"><br><br><button class="btn" onclick="compare()">قارن</button><div id="res"></div></div>`;
main.innerHTML=h}
function send(){let x=document.getElementById('chatin').value.trim();if(!x)return;S.chat.push({who:'me',t:x},{who:'ai',t:'وصلت فكرتك. في النسخة القادمة سأستخدم ذاكرتك وأهدافك لصياغة رد أكثر تخصيصاً.'});go('chat')}
function addMem(){let x=memin.value.trim();if(x){S.mem.push(x);save();go('memory')}}function delMem(i){S.mem.splice(i,1);save();go('memory')}
function addGoal(){let x=goin.value.trim();if(x){S.goals.push({t:x,done:false});save();go('goals')}}function delGoal(i){S.goals.splice(i,1);save();go('goals')}function toggleGoal(i){S.goals[i].done=!S.goals[i].done;save();go('goals')}
function compare(){let a=document.getElementById('a').value,b=document.getElementById('b').value;document.getElementById('res').innerHTML=`<div class="card"><h3>المقارنة</h3><p class="muted">الخياران: <b>${a||'—'}</b> و <b>${b||'—'}</b>. أضف معايير مثل السعر، الوقت، المخاطر أو الراحة لتحصل على مقارنة مفيدة.</p></div>`}
go('home');
</script></body></html>'''
(root/"index.html").write_text(index,encoding="utf-8")
(root/"manifest.json").write_text(json.dumps({
"name":"MINE — نسخة مني","short_name":"MINE","start_url":"./index.html","display":"standalone",
"background_color":"#0b0e13","theme_color":"#10131a","lang":"ar","dir":"rtl",
"icons":[]
},ensure_ascii=False,indent=2),encoding="utf-8")
(root/"README.txt").write_text("""MINE — نسخة مني
نسخة تجريبية PWA تعمل من الهاتف.

التثبيت:
1) ارفع هذا المجلد إلى استضافة صفحات ثابتة HTTPS.
2) افتح الرابط من Chrome على Android.
3) من قائمة المتصفح اختر "إضافة إلى الشاشة الرئيسية" أو "تثبيت التطبيق".

البيانات التجريبية (الذاكرة والأهداف) تُحفظ محلياً في المتصفح.
هذه النسخة لا تحتوي بعد على API حقيقي للذكاء الاصطناعي.
""",encoding="utf-8")

zip_path=Path("/mnt/data/MINE_Test_Version.zip")
if zip_path.exists(): zip_path.unlink()
with zipfile.ZipFile(zip_path,"w",zipfile.ZIP_DEFLATED) as z:
    for f in root.rglob("*"):
        z.write(f,f.relative_to(root))
print(zip_path)
# mine-app
