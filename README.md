
<!DOCTYPE html>
<html lang="en">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1">
<title>SOULBOUND: Beneath the Mountain</title>
<style>
*{box-sizing:border-box}
body{margin:0;background:#09070d;color:#fff;font-family:Consolas,"Courier New",monospace;display:flex;justify-content:center;padding:16px}
#game{width:min(940px,100%);background:#000;border:4px solid #eee;padding:16px;box-shadow:0 0 30px #33223e}
#top{display:flex;justify-content:space-between;gap:10px;flex-wrap:wrap;font-size:14px;margin-bottom:12px}
canvas{width:100%;height:auto;display:block;border:2px solid #eee;background:#000;image-rendering:pixelated}
#dialogue{white-space:pre-wrap;line-height:1.5;min-height:105px;border:2px solid #eee;padding:12px;margin-top:12px}
#actions{display:grid;grid-template-columns:repeat(4,minmax(0,1fr));gap:8px;margin-top:10px}
button{font:bold 15px Consolas,monospace;padding:12px 4px;background:#000;color:#ffbd35;border:2px solid #ffbd35;cursor:pointer}
button:hover:not(:disabled){background:#ffbd35;color:#000}
button:disabled{opacity:.28;cursor:not-allowed}
#hint{font-size:12px;color:#aaa;line-height:1.5;margin-top:12px}
#hpbar{display:inline-block;width:90px;height:12px;border:1px solid white;background:#461217;vertical-align:middle}
#hpfill{display:block;height:100%;background:#ff3038;width:100%}
@media(max-width:560px){
 #game{padding:8px}
 #dialogue{font-size:13px}
 button{font-size:11px;padding:10px 2px}
 #top{font-size:11px}
}
</style>
</head>
<body>
<main id="game">
 <div id="top">
  <span id="chapter">SOULBOUND | PROLOGUE</span>
  <span>LV <b id="lv">1</b> · HP <b id="hp">24</b>/<span id="maxhp">24</span> <span id="hpbar"><span id="hpfill"></span></span></span>
  <span id="gold">GOLD 0</span>
 </div>
 <canvas id="screen" width="800" height="430"></canvas>
 <div id="dialogue" role="status" aria-live="polite"></div>
 <div id="actions">
  <button id="fight">⚔ FIGHT</button>
  <button id="act">☼ ACT</button>
  <button id="item">♥ ITEM</button>
  <button id="mercy">♡ MERCY</button>
 </div>
 <div id="hint">
  Story: press ENTER or CONTINUE.
  Battle rule: FIGHT or ACT is randomly selected for each player turn.
  The other action is locked.
  Use ITEM to heal and MERCY when available.
  Dodge with WASD or ARROW KEYS.
  Press SPACE during a FIGHT meter.
  Press R after an ending or game over.
 </div>
</main>

<script>
(()=>{
const canvas=document.querySelector('#screen');
const ctx=canvas.getContext('2d');
const $=id=>document.getElementById(id);
const W=800,H=430;
const arena={x:220,y:245,w:360,h:145};

let state='story',scene='prologue',line=0,frame=0,last=0,keys={};
let hp=24,maxHp=24,gold=0,lv=1,xp=0,kills=0,mercyCount=0;
let enemy={name:'',kind:'slime',color:'#76d58a'};
let enemyHp=0,enemyMax=0,mood=0,turn=0;
let allowedAction='fight';
let soul={x:400,y:320},bullets=[],invuln=0;
let meter=0,meterDir=1,ending='';
let inventory=['Starberry','Starberry'];
let flags={key:false,metPip:false,bridge:false};

const chapters={
prologue:[
'Long before your story began, humans and monsters lived beneath the same sky.',
'Monsters could shape magic from their SOULS. Humans feared what they did not understand.',
'A war shattered the peace. Humans won, and the monsters were driven into the mountain.',
'Seven mages sealed the mountain with a barrier. No monster could cross it.',
'Generations passed. The war became a legend, and the mountain became a warning.',
'“Climb the mountain, and you may never come home.”',
'You followed a trail of golden flowers to the summit. The ground cracked beneath your feet.',
'You fell through the darkness. Wind roared in your ears, then—silence.',
'You wake in a bed of golden flowers under a ceiling of black stone.',
'There is no ladder. No phone signal. Only a narrow passage leading deeper underground.',
'Press ENTER or CONTINUE to begin your journey.'
],
flower:[
'A flower sways in the still air. Its petals are bright, but its smile feels practiced.',
'FLOWER: “Oh, hey! A human! You look lost.”',
'FLOWER: “Down here, your SOUL is the most important thing you have.”',
'It draws a tiny red heart in the dust. “That little heart is you. Keep it safe.”',
'FLOWER: “Let me teach you the rules. Everyone gets a turn, and everyone gets hurt.”',
'For one instant, its eyes turn flat and cold.',
'FLOWER: “Let’s see how you handle a little danger.”'
],
rootHall:[
'The flower disappears through a crack in the stone before you can ask its name.',
'You enter ROOT HALL, a maze of old pillars and roots thick as tree trunks.',
'Small lanterns burn with blue flame. Someone has been keeping them lit.',
'On a wall, you find a faded mural: a human child standing beside a monster child.',
'Someone has scratched out the human’s face.',
'A wooden sign points toward the RUINS. Another points toward a locked side door.',
'The door has a keyhole shaped like a star. You make a note to look for a key.'
],
ruins:[
'The ruins smell of rain even though no rain has fallen here in centuries.',
'A nervous little monster blocks the bridge. Its body looks like a soft green cloud.',
'CRITTER: “I was told to guard this bridge, but I forgot what I was guarding it from.”',
'It glances at your hands. “You are not going to hurt me, are you?”',
'You can only perform the action randomly assigned at the start of each turn.'
],
afterRuins:[
'The critter steps aside, relieved. It leaves a few coins beside the path.',
'You cross the bridge into a tunnel lined with cracked mirrors.',
'In each reflection, you see yourself a little older—walking, running, standing before a throne.',
'At the tunnel’s end, a sign reads: “THE ECHO GALLERY. DO NOT BELIEVE EVERYTHING YOU HEAR.”',
'From somewhere ahead, you hear a skeleton laughing at his own joke.'
],
echo:[
'Two skeletons stand beneath a chandelier made from old spoons.',
'MAGNUS: “HUMAN! I, MAGNUS, HAVE FINALLY FOUND YOU!”',
'PIP: “you found a broom closet, actually. the human is over there.”',
'MAGNUS: “I WAS TESTING THE CLOSET.”',
'Pip turns to you. “Sorry. My brother has a lot of confidence and not a lot of map skills.”',
'Magnus challenges you to a dramatic duel, then trips over his own cape.',
'Pip quietly slips you a note: “The king is scared. Don’t let anyone tell you otherwise.”',
'They point you toward the underground city. Magnus insists he is escorting you, then walks backward into a wall.'
],
mushroom:[
'You enter the MUSHROOM GROVE, where glowing caps light a forest of stone.',
'Every mushroom hums a different note. Together they make a strange, beautiful song.',
'A tiny vendor named NIM sits beside a cart of bottled starlight.',
'NIM: “Travelers pay with gold, stories, or secrets. You look short on all three.”',
'Nim gives you a Starberry anyway. “Pay it forward, little wanderer.”',
'A deep rumble shakes dust from the ceiling. Something enormous moves beneath the grove.'
],
miniboss:[
'The tunnel opens into a circular chamber. A stone guardian rises from the floor.',
'GUARDIAN: “ONLY THOSE WHO KNOW MERCY MAY PASS.”',
'Its eyes glow like two old stars. It does not seem eager to fight.',
'The guardian lowers its head, waiting to see what you will do.'
],
city:[
'The underground city of LANTERNFALL stretches around a lake of glowing blue water.',
'Families gather under paper lanterns. A band plays on a bridge. A child chases a floating orb.',
'Some monsters stare at you with fear. Others offer a wave, as if they have been waiting for you.',
'An old shopkeeper hands you a Starberry. “The king has called for a human. He believes your SOUL can break the barrier.”',
'You hear another rumor: the barrier may also be broken by a choice freely given.',
'You cross the market square toward the royal lift.'
],
archives:[
'The royal lift stops at the ARCHIVES, a library carved into the mountain’s heart.',
'Books fill the shelves from floor to ceiling. Some are written in human script, others in monster runes.',
'You find the journal of Queen Elara, who died long before you were born.',
'JOURNAL: “A barrier can be broken by power. But power is not the same as freedom.”',
'JOURNAL: “If a human and monster SOUL choose to trust one another, the seal may open without a sacrifice.”',
'The final page has been torn out. A golden key is tucked into the binding.',
'You take the key. Somewhere below, a bell rings three times.'
],
train:[
'The lift opens onto an underground railway platform. A train waits in a cloud of steam.',
'Pip is there, breathless. “the castle bridge is broken. this train is the only way across the chasm.”',
'Inside, the seats are empty. A pale figure is slumped by the far door.',
'The figure raises one trembling hand. “Please... help me...”',
'You take the stranger’s hand. Their fingers are ice-cold.',
'STRANGER: “I thought I could make it...” Their body shudders. Dark veins spread beneath their skin.',
'Pip backs away. “that’s not a person anymore. get ready!”',
'The stranger collapses, then rises as a monster. The train doors slam shut.'
],
trainBattle:[
'The infected passenger lunges. More creatures claw their way out of the dark carriages.',
'You fight your way through the train while it rattles over the chasm.',
'At last, the brakes scream. The doors slide open onto the castle’s outer platform.',
'Pip helps you down. “I’ll get the others to safety. you need to reach the king.”',
'Across the tracks, the royal gates begin to open.'
],
castle:[
'The castle is silent. Its halls are lined with portraits of monsters who never saw the surface.',
'At the throne room door, you find a small wooden toy horse. Someone has repaired its broken leg.',
'KING AURON stands beside a window overlooking the barrier: a shimmering dome of pale light.',
'AURON: “I have watched my people grow old beneath a sky they cannot touch.”',
'AURON: “Your SOUL could break the barrier. I will not pretend the price is small.”',
'AURON: “But I have promised them a future. I do not know how to keep that promise any other way.”',
'He removes his crown and places it on the throne.',
'AURON: “Show me what kind of human you are.”'
],
epilogue:[
'The battle ends. The king looks at you—not as a weapon, but as a child who fell into a terrible story.',
'Pip, Magnus, Nim, and the monsters of Lanternfall gather in the throne room.',
'The archive journal glows. Its missing final page appears in a shower of golden dust.',
'The page describes a third path: two SOULS, freely offered, can change the barrier instead of destroying it.',
'You reach out your hand. The king takes it.',
'Light flows across the mountain. The barrier opens like a curtain at dawn.',
'For the first time in centuries, monsters see the open sky.',
'Some cry. Some laugh. Magnus declares that he is “definitely not crying,” while Pip pats his shoulder.',
'Your journey is not over—but now, the world above is waiting.',
'THE END — Press R to play again.'
]
};

function say(t){$('dialogue').textContent=t}
function hud(){
 $('hp').textContent=hp;
 $('maxhp').textContent=maxHp;
 $('hpfill').style.width=(hp/maxHp*100)+'%';
 $('gold').textContent='GOLD '+gold;
 $('lv').textContent=lv;
}
function rand(a,b){return a+Math.random()*(b-a)}
function txt(t,x,y,size=20,color='#fff',align='center'){
 ctx.fillStyle=color;
 ctx.font=`bold ${size}px Consolas,monospace`;
 ctx.textAlign=align;
 ctx.fillText(t,x,y);
}
function heart(x,y,s=1,color='#ff3038'){
 ctx.save();ctx.translate(x,y);ctx.scale(s,s);ctx.fillStyle=color;
 ctx.beginPath();ctx.moveTo(0,7);
 ctx.bezierCurveTo(-19,-5,-11,-17,0,-8);
 ctx.bezierCurveTo(11,-17,19,-5,0,7);
 ctx.fill();ctx.restore();
}
function human(x,y,c='#c7a2ff'){
 ctx.fillStyle=c;ctx.fillRect(x-10,y-17,20,24);
 ctx.fillStyle='#f0c8a8';ctx.fillRect(x-8,y-31,16,15);
 ctx.fillStyle='#38284b';ctx.fillRect(x-9,y-34,18,6);
 ctx.fillStyle=c;ctx.fillRect(x-9,y+7,7,15);ctx.fillRect(x+2,y+7,7,15);
}
function monsterArt(){
 let bob=Math.sin(frame*.06)*4;
 if(enemy.kind==='flower'){
  ctx.fillStyle='#318d48';ctx.fillRect(390,120,20,70);
  for(let i=0;i<8;i++){
   ctx.fillStyle=i%2?'#f4d44b':'#e64c65';
   ctx.beginPath();
   ctx.ellipse(400+Math.cos(i*Math.PI/4)*30,110+Math.sin(i*Math.PI/4)*30,18,12,i*Math.PI/4,0,Math.PI*2);
   ctx.fill();
  }
  ctx.fillStyle='#e6bd38';ctx.beginPath();ctx.arc(400,110,22,0,Math.PI*2);ctx.fill();
  ctx.fillStyle='#111';ctx.fillRect(392,105,5,7);ctx.fillRect(404,105,5,7);ctx.fillRect(392,119,16,3);
 }else if(enemy.kind==='slime'||enemy.kind==='critter'){
  ctx.save();ctx.translate(400,145+bob);ctx.fillStyle=enemy.color;ctx.strokeStyle='#fff';ctx.lineWidth=3;
  ctx.beginPath();ctx.ellipse(0,0,78,56,0,0,Math.PI*2);ctx.fill();ctx.stroke();
  ctx.fillStyle='#fff';ctx.beginPath();
  ctx.ellipse(-25,-8,18,22,0,0,Math.PI*2);ctx.ellipse(25,-8,18,22,0,0,Math.PI*2);ctx.fill();
  ctx.fillStyle='#111';ctx.fillRect(-23,-7,5,12);ctx.fillRect(26,-7,5,12);ctx.restore();
 }else if(enemy.kind==='guardian'){
  ctx.fillStyle='#817b91';ctx.fillRect(335,80,130,120);
  ctx.fillStyle='#fff';ctx.fillRect(355,105,25,15);ctx.fillRect(420,105,25,15);
  ctx.fillStyle='#ffda72';ctx.fillRect(363,108,10,9);ctx.fillRect(428,108,10,9);
  ctx.fillStyle='#393244';ctx.fillRect(365,155,70,9);
  ctx.fillStyle='#a9a2b7';ctx.fillRect(315,110,22,80);ctx.fillRect(463,110,22,80);
 }else if(enemy.kind==='king'){
  ctx.fillStyle=enemy.color;ctx.fillRect(355,95,90,100);
  ctx.fillStyle='#fff';ctx.fillRect(370,115,18,23);ctx.fillRect(412,115,18,23);
  ctx.fillStyle='#111';ctx.fillRect(378,121,6,12);ctx.fillRect(420,121,6,12);
  ctx.fillStyle='#d9b98b';ctx.fillRect(345,85,110,12);
  ctx.fillStyle='#fff';ctx.fillRect(380,160,40,5);
 }else{
  ctx.fillStyle=enemy.color;ctx.fillRect(355,95,90,100);
  ctx.fillStyle='#e7e2f5';ctx.fillRect(370,115,18,23);ctx.fillRect(412,115,18,23);
  ctx.fillStyle='#e33';ctx.fillRect(378,121,6,12);ctx.fillRect(420,121,6,12);
  ctx.fillStyle='#4d263e';ctx.fillRect(380,160,40,5);
 }
}
function drawScene(){
 ctx.fillStyle='#000';ctx.fillRect(0,0,W,H);
 for(let i=0;i<48;i++){
  ctx.fillStyle=i%3?'#25202b':'#42354b';
  ctx.fillRect((i*137)%W,(i*71)%H,3,3);
 }
 if(scene==='prologue'||scene==='flower'){
  ctx.fillStyle='#24152e';ctx.fillRect(0,260,W,170);
  for(let i=0;i<8;i++){
   ctx.fillStyle='#b7a04e';ctx.beginPath();
   ctx.ellipse(75+i*95,280+(i%2)*13,17,6,i*.3,0,Math.PI*2);ctx.fill();
  }
  human(400,240);
  if(scene==='flower'){
   ctx.fillStyle='#55bb55';ctx.beginPath();ctx.arc(530,205,40,0,Math.PI*2);ctx.fill();
   ctx.fillStyle='#fff';ctx.fillRect(515,190,7,9);ctx.fillRect(537,190,7,9);
  }
 }else if(['rootHall','ruins','afterRuins','miniboss','echo','mushroom','archives','train','trainBattle'].includes(scene)){
  ctx.fillStyle=scene==='mushroom'?'#152b2c':scene==='train'||scene==='trainBattle'?'#20232c':'#25202d';
  ctx.fillRect(0,240,W,190);
  for(let i=0;i<9;i++){
   ctx.fillStyle=scene==='mushroom'?'#285050':'#493a53';
   ctx.fillRect(i*100,180-(i%3)*15,80,100);
  }
  if(scene==='mushroom'){
   for(let i=0;i<7;i++){
    ctx.fillStyle=i%2?'#a45bd2':'#4bcac0';
    ctx.beginPath();ctx.arc(60+i*110,260-(i%3)*30,15,Math.PI,0);ctx.fill();
    ctx.fillRect(55+i*110,260-(i%3)*30,10,24);
   }
  }
  if(scene==='train'||scene==='trainBattle'){
   ctx.fillStyle='#3d414b';ctx.fillRect(40,90,720,170);
   ctx.strokeStyle='#aaa';ctx.lineWidth=4;
   ctx.strokeRect(60,105,145,95);ctx.strokeRect(230,105,145,95);
   ctx.strokeRect(400,105,145,95);ctx.strokeRect(570,105,145,95);
   if(scene==='trainBattle')txt('!!',400,70,35,'#ff4242');
  }
  if(scene==='echo'){txt('☠',485,200,65,'#fff');txt('☠',560,220,45,'#fff')}
  if(scene==='miniboss')txt('✦',400,140,70,'#ffd86c');
 }else if(scene==='city'){
  ctx.fillStyle='#15233b';ctx.fillRect(0,180,W,250);
  for(let i=0;i<8;i++){
   ctx.fillStyle='#293c60';ctx.fillRect(i*105,130+(i%2)*30,85,170);
   ctx.fillStyle='#9ee6ff';ctx.fillRect(i*105+20,155+(i%2)*30,10,18);
   ctx.fillRect(i*105+50,190+(i%2)*30,10,18);
  }
 }else if(scene==='castle'||scene==='epilogue'){
  ctx.fillStyle='#28212b';ctx.fillRect(0,160,W,270);
  ctx.fillStyle='#77623e';ctx.fillRect(90,90,620,280);
  ctx.fillStyle='#17131b';ctx.fillRect(310,150,180,220);
  txt('♛',400,215,60,'#ffd66b');human(250,280);
  if(scene==='epilogue'){
   ctx.fillStyle='#ffdf7e';ctx.globalAlpha=.5+Math.sin(frame*.08)*.3;
   ctx.fillRect(0,0,W,H);ctx.globalAlpha=1;
  }
 }
 if(state==='ending')txt('THE END',400,365,34,'#ffdf7e');
}
function draw(){
 if(['story','ending'].includes(state)){drawScene();return}
 ctx.fillStyle='#000';ctx.fillRect(0,0,W,H);
 for(let i=0;i<36;i++){
  ctx.fillStyle='#201c28';ctx.fillRect((i*97)%W,(i*53)%H,3,3);
 }
 monsterArt();txt(enemy.name,400,218,19);
 ctx.strokeStyle='#fff';ctx.lineWidth=3;ctx.strokeRect(arena.x,arena.y,arena.w,arena.h);

 if(state==='attack'){
  for(const b of bullets){
   ctx.fillStyle=b.color||'#fff';ctx.fillRect(b.x,b.y,b.s,b.s);
  }
  if(invuln<=0||Math.floor(frame/3)%2===0)heart(soul.x,soul.y,.8);
  txt('DODGE!',400,235,14,'#bbb');
 }else if(state==='fight'){
  ctx.strokeRect(250,310,300,22);
  ctx.fillStyle='#ffbd35';ctx.fillRect(255+meter*2.85,313,6,16);
  txt('SPACE: STRIKE',400,365,14);
 }else if(state==='win'){
  txt('VICTORY',400,300,32,'#ffdf7e');
 }else if(state==='dead'){
  ctx.fillStyle='#000';ctx.fillRect(0,0,W,H);
  heart(400,170,2);txt('GAME OVER',400,245,36,'#ff3038');
  txt('Press R to restart',400,285,20);
 }
}
function updateButtons(){
 const story=state==='story';
 $('fight').textContent=story?'CONTINUE':allowedAction==='fight'?'⚔ FIGHT (CHOSEN)':'⚔ FIGHT (LOCKED)';
 $('act').textContent=story?'CONTINUE':allowedAction==='act'?'☼ ACT (CHOSEN)':'☼ ACT (LOCKED)';
 $('item').textContent='♥ ITEM ('+inventory.length+')';
 $('mercy').textContent=state==='win'?'CONTINUE':story?'CONTINUE':'♡ MERCY';
 $('fight').disabled=story?false:state!=='menu'||allowedAction!=='fight';
 $('act').disabled=story?false:state!=='menu'||allowedAction!=='act';
 $('item').disabled=story?false:state!=='menu'||inventory.length===0;
 $('mercy').disabled=story?false:!(state==='menu'||state==='win');
}
function setStory(key){
 scene=key;state='story';line=0;
 $('chapter').textContent='SOULBOUND | '+key.toUpperCase();
 say(chapters[key][0]);updateButtons();
}
function nextStory(){
 if(state!=='story')return;
 line++;
 if(line>=chapters[scene].length){storyDone();return}
 say(chapters[scene][line]);
}
function storyDone(){
 if(scene==='prologue'){setStory('flower');return}
 if(scene==='flower'){startBattle('SMILING FLOWER','flower',22,'#e7bf35');return}
 if(scene==='rootHall'){setStory('ruins');return}
 if(scene==='ruins'){startBattle('SHY CAVE CRITTER','critter',26,'#76d58a');return}
 if(scene==='afterRuins'){setStory('echo');return}
 if(scene==='echo'){flags.metPip=true;setStory('mushroom');return}
 if(scene==='mushroom'){startBattle('TUNNEL STALKER','stalker',30,'#8d7ec6');return}
 if(scene==='miniboss'){startBattle('ANCIENT GUARDIAN','guardian',42,'#817b91');return}
 if(scene==='city'){setStory('archives');return}
 if(scene==='archives'){flags.key=true;setStory('train');return}
 if(scene==='train'){startBattle('INFECTED PASSENGER','infected',38,'#7b8d78');return}
 if(scene==='trainBattle'){setStory('castle');return}
 if(scene==='castle'){startBattle('KING AURON','king',72,'#c2a26b');return}
 if(scene==='epilogue'){
  state='ending';ending='PEACE';
  say('TRUE PEACE ENDING\nYou and the monsters step into a new world together.\n\nPress R to begin again.');
  updateButtons();
 }
}
function startBattle(name,kind,amount,color){
 enemy={name,kind,color};enemyHp=enemyMax=amount;mood=0;turn=0;
 state='menu';scene='battle';
 $('chapter').textContent='SOULBOUND | ENCOUNTER';
 chooseAction();
 say(name+' blocks your path.\nThe next action is chosen by fate...');
 updateButtons();
}
function chooseAction(){
 allowedAction=Math.random()<.5?'fight':'act';
 $('fight').textContent=allowedAction==='fight'?'⚔ FIGHT (CHOSEN)':'⚔ FIGHT (LOCKED)';

<!DOCTYPE html>
<html lang="en">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1">
<title>SOULBOUND: Beneath the Mountain</title>
<style>
*{box-sizing:border-box}
body{margin:0;background:#09070d;color:#fff;font-family:Consolas,"Courier New",monospace;display:flex;justify-content:center;padding:16px}
#game{width:min(940px,100%);background:#000;border:4px solid #eee;padding:16px;box-shadow:0 0 30px #33223e}
#top{display:flex;justify-content:space-between;gap:10px;flex-wrap:wrap;font-size:14px;margin-bottom:12px}
canvas{width:100%;height:auto;display:block;border:2px solid #eee;background:#000;image-rendering:pixelated}
#dialogue{white-space:pre-wrap;line-height:1.5;min-height:105px;border:2px solid #eee;padding:12px;margin-top:12px}
#actions{display:grid;grid-template-columns:repeat(4,minmax(0,1fr));gap:8px;margin-top:10px}
button{font:bold 15px Consolas,monospace;padding:12px 4px;background:#000;color:#ffbd35;border:2px solid #ffbd35;cursor:pointer}
button:hover:not(:disabled){background:#ffbd35;color:#000}
button:disabled{opacity:.28;cursor:not-allowed}
#hint{font-size:12px;color:#aaa;line-height:1.5;margin-top:12px}
#hpbar{display:inline-block;width:90px;height:12px;border:1px solid white;background:#461217;vertical-align:middle}
#hpfill{display:block;height:100%;background:#ff3038;width:100%}
@media(max-width:560px){
 #game{padding:8px}
 #dialogue{font-size:13px}
 button{font-size:11px;padding:10px 2px}
 #top{font-size:11px}
}
</style>
</head>
<body>
<main id="game">
 <div id="top">
  <span id="chapter">SOULBOUND | PROLOGUE</span>
  <span>LV <b id="lv">1</b> · HP <b id="hp">24</b>/<span id="maxhp">24</span> <span id="hpbar"><span id="hpfill"></span></span></span>
  <span id="gold">GOLD 0</span>
 </div>
 <canvas id="screen" width="800" height="430"></canvas>
 <div id="dialogue" role="status" aria-live="polite"></div>
 <div id="actions">
  <button id="fight">⚔ FIGHT</button>
  <button id="act">☼ ACT</button>
  <button id="item">♥ ITEM</button>
  <button id="mercy">♡ MERCY</button>
 </div>
 <div id="hint">
  Story: press ENTER or CONTINUE.
  Battle rule: FIGHT or ACT is randomly selected for each player turn.
  The other action is locked.
  Use ITEM to heal and MERCY when available.
  Dodge with WASD or ARROW KEYS.
  Press SPACE during a FIGHT meter.
  Press R after an ending or game over.
 </div>
</main>

<script>
(()=>{
const canvas=document.querySelector('#screen');
const ctx=canvas.getContext('2d');
const $=id=>document.getElementById(id);
const W=800,H=430;
const arena={x:220,y:245,w:360,h:145};

let state='story',scene='prologue',line=0,frame=0,last=0,keys={};
let hp=24,maxHp=24,gold=0,lv=1,xp=0,kills=0,mercyCount=0;
let enemy={name:'',kind:'slime',color:'#76d58a'};
let enemyHp=0,enemyMax=0,mood=0,turn=0;
let allowedAction='fight';
let soul={x:400,y:320},bullets=[],invuln=0;
let meter=0,meterDir=1,ending='';
let inventory=['Starberry','Starberry'];
let flags={key:false,metPip:false,bridge:false};

const chapters={
prologue:[
'Long before your story began, humans and monsters lived beneath the same sky.',
'Monsters could shape magic from their SOULS. Humans feared what they did not understand.',
'A war shattered the peace. Humans won, and the monsters were driven into the mountain.',
'Seven mages sealed the mountain with a barrier. No monster could cross it.',
'Generations passed. The war became a legend, and the mountain became a warning.',
'“Climb the mountain, and you may never come home.”',
'You followed a trail of golden flowers to the summit. The ground cracked beneath your feet.',
'You fell through the darkness. Wind roared in your ears, then—silence.',
'You wake in a bed of golden flowers under a ceiling of black stone.',
'There is no ladder. No phone signal. Only a narrow passage leading deeper underground.',
'Press ENTER or CONTINUE to begin your journey.'
],
flower:[
'A flower sways in the still air. Its petals are bright, but its smile feels practiced.',
'FLOWER: “Oh, hey! A human! You look lost.”',
'FLOWER: “Down here, your SOUL is the most important thing you have.”',
'It draws a tiny red heart in the dust. “That little heart is you. Keep it safe.”',
'FLOWER: “Let me teach you the rules. Everyone gets a turn, and everyone gets hurt.”',
'For one instant, its eyes turn flat and cold.',
'FLOWER: “Let’s see how you handle a little danger.”'
],
rootHall:[
'The flower disappears through a crack in the stone before you can ask its name.',
'You enter ROOT HALL, a maze of old pillars and roots thick as tree trunks.',
'Small lanterns burn with blue flame. Someone has been keeping them lit.',
'On a wall, you find a faded mural: a human child standing beside a monster child.',
'Someone has scratched out the human’s face.',
'A wooden sign points toward the RUINS. Another points toward a locked side door.',
'The door has a keyhole shaped like a star. You make a note to look for a key.'
],
ruins:[
'The ruins smell of rain even though no rain has fallen here in centuries.',
'A nervous little monster blocks the bridge. Its body looks like a soft green cloud.',
'CRITTER: “I was told to guard this bridge, but I forgot what I was guarding it from.”',
'It glances at your hands. “You are not going to hurt me, are you?”',
'You can only perform the action randomly assigned at the start of each turn.'
],
afterRuins:[
'The critter steps aside, relieved. It leaves a few coins beside the path.',
'You cross the bridge into a tunnel lined with cracked mirrors.',
'In each reflection, you see yourself a little older—walking, running, standing before a throne.',
'At the tunnel’s end, a sign reads: “THE ECHO GALLERY. DO NOT BELIEVE EVERYTHING YOU HEAR.”',
'From somewhere ahead, you hear a skeleton laughing at his own joke.'
],
echo:[
'Two skeletons stand beneath a chandelier made from old spoons.',
'MAGNUS: “HUMAN! I, MAGNUS, HAVE FINALLY FOUND YOU!”',
'PIP: “you found a broom closet, actually. the human is over there.”',
'MAGNUS: “I WAS TESTING THE CLOSET.”',
'Pip turns to you. “Sorry. My brother has a lot of confidence and not a lot of map skills.”',
'Magnus challenges you to a dramatic duel, then trips over his own cape.',
'Pip quietly slips you a note: “The king is scared. Don’t let anyone tell you otherwise.”',
'They point you toward the underground city. Magnus insists he is escorting you, then walks backward into a wall.'
],
mushroom:[
'You enter the MUSHROOM GROVE, where glowing caps light a forest of stone.',
'Every mushroom hums a different note. Together they make a strange, beautiful song.',
'A tiny vendor named NIM sits beside a cart of bottled starlight.',
'NIM: “Travelers pay with gold, stories, or secrets. You look short on all three.”',
'Nim gives you a Starberry anyway. “Pay it forward, little wanderer.”',
'A deep rumble shakes dust from the ceiling. Something enormous moves beneath the grove.'
],
miniboss:[
'The tunnel opens into a circular chamber. A stone guardian rises from the floor.',
'GUARDIAN: “ONLY THOSE WHO KNOW MERCY MAY PASS.”',
'Its eyes glow like two old stars. It does not seem eager to fight.',
'The guardian lowers its head, waiting to see what you will do.'
],
city:[
'The underground city of LANTERNFALL stretches around a lake of glowing blue water.',
'Families gather under paper lanterns. A band plays on a bridge. A child chases a floating orb.',
'Some monsters stare at you with fear. Others offer a wave, as if they have been waiting for you.',
'An old shopkeeper hands you a Starberry. “The king has called for a human. He believes your SOUL can break the barrier.”',
'You hear another rumor: the barrier may also be broken by a choice freely given.',
'You cross the market square toward the royal lift.'
],
archives:[
'The royal lift stops at the ARCHIVES, a library carved into the mountain’s heart.',
'Books fill the shelves from floor to ceiling. Some are written in human script, others in monster runes.',
'You find the journal of Queen Elara, who died long before you were born.',
'JOURNAL: “A barrier can be broken by power. But power is not the same as freedom.”',
'JOURNAL: “If a human and monster SOUL choose to trust one another, the seal may open without a sacrifice.”',
'The final page has been torn out. A golden key is tucked into the binding.',
'You take the key. Somewhere below, a bell rings three times.'
],
train:[
'The lift opens onto an underground railway platform. A train waits in a cloud of steam.',
'Pip is there, breathless. “the castle bridge is broken. this train is the only way across the chasm.”',
'Inside, the seats are empty. A pale figure is slumped by the far door.',
'The figure raises one trembling hand. “Please... help me...”',
'You take the stranger’s hand. Their fingers are ice-cold.',
'STRANGER: “I thought I could make it...” Their body shudders. Dark veins spread beneath their skin.',
'Pip backs away. “that’s not a person anymore. get ready!”',
'The stranger collapses, then rises as a monster. The train doors slam shut.'
],
trainBattle:[
'The infected passenger lunges. More creatures claw their way out of the dark carriages.',
'You fight your way through the train while it rattles over the chasm.',
'At last, the brakes scream. The doors slide open onto the castle’s outer platform.',
'Pip helps you down. “I’ll get the others to safety. you need to reach the king.”',
'Across the tracks, the royal gates begin to open.'
],
castle:[
'The castle is silent. Its halls are lined with portraits of monsters who never saw the surface.',
'At the throne room door, you find a small wooden toy horse. Someone has repaired its broken leg.',
'KING AURON stands beside a window overlooking the barrier: a shimmering dome of pale light.',
'AURON: “I have watched my people grow old beneath a sky they cannot touch.”',
'AURON: “Your SOUL could break the barrier. I will not pretend the price is small.”',
'AURON: “But I have promised them a future. I do not know how to keep that promise any other way.”',
'He removes his crown and places it on the throne.',
'AURON: “Show me what kind of human you are.”'
],
epilogue:[
'The battle ends. The king looks at you—not as a weapon, but as a child who fell into a terrible story.',
'Pip, Magnus, Nim, and the monsters of Lanternfall gather in the throne room.',
'The archive journal glows. Its missing final page appears in a shower of golden dust.',
'The page describes a third path: two SOULS, freely offered, can change the barrier instead of destroying it.',
'You reach out your hand. The king takes it.',
'Light flows across the mountain. The barrier opens like a curtain at dawn.',
'For the first time in centuries, monsters see the open sky.',
'Some cry. Some laugh. Magnus declares that he is “definitely not crying,” while Pip pats his shoulder.',
'Your journey is not over—but now, the world above is waiting.',
'THE END — Press R to play again.'
]
};

function say(t){$('dialogue').textContent=t}
function hud(){
 $('hp').textContent=hp;
 $('maxhp').textContent=maxHp;
 $('hpfill').style.width=(hp/maxHp*100)+'%';
 $('gold').textContent='GOLD '+gold;
 $('lv').textContent=lv;
}
function rand(a,b){return a+Math.random()*(b-a)}
function txt(t,x,y,size=20,color='#fff',align='center'){
 ctx.fillStyle=color;
 ctx.font=`bold ${size}px Consolas,monospace`;
 ctx.textAlign=align;
 ctx.fillText(t,x,y);
}
function heart(x,y,s=1,color='#ff3038'){
 ctx.save();ctx.translate(x,y);ctx.scale(s,s);ctx.fillStyle=color;
 ctx.beginPath();ctx.moveTo(0,7);
 ctx.bezierCurveTo(-19,-5,-11,-17,0,-8);
 ctx.bezierCurveTo(11,-17,19,-5,0,7);
 ctx.fill();ctx.restore();
}
function human(x,y,c='#c7a2ff'){
 ctx.fillStyle=c;ctx.fillRect(x-10,y-17,20,24);
 ctx.fillStyle='#f0c8a8';ctx.fillRect(x-8,y-31,16,15);
 ctx.fillStyle='#38284b';ctx.fillRect(x-9,y-34,18,6);
 ctx.fillStyle=c;ctx.fillRect(x-9,y+7,7,15);ctx.fillRect(x+2,y+7,7,15);
}
function monsterArt(){
 let bob=Math.sin(frame*.06)*4;
 if(enemy.kind==='flower'){
  ctx.fillStyle='#318d48';ctx.fillRect(390,120,20,70);
  for(let i=0;i<8;i++){
   ctx.fillStyle=i%2?'#f4d44b':'#e64c65';
   ctx.beginPath();
   ctx.ellipse(400+Math.cos(i*Math.PI/4)*30,110+Math.sin(i*Math.PI/4)*30,18,12,i*Math.PI/4,0,Math.PI*2);
   ctx.fill();
  }
  ctx.fillStyle='#e6bd38';ctx.beginPath();ctx.arc(400,110,22,0,Math.PI*2);ctx.fill();
  ctx.fillStyle='#111';ctx.fillRect(392,105,5,7);ctx.fillRect(404,105,5,7);ctx.fillRect(392,119,16,3);
 }else if(enemy.kind==='slime'||enemy.kind==='critter'){
  ctx.save();ctx.translate(400,145+bob);ctx.fillStyle=enemy.color;ctx.strokeStyle='#fff';ctx.lineWidth=3;
  ctx.beginPath();ctx.ellipse(0,0,78,56,0,0,Math.PI*2);ctx.fill();ctx.stroke();
  ctx.fillStyle='#fff';ctx.beginPath();
  ctx.ellipse(-25,-8,18,22,0,0,Math.PI*2);ctx.ellipse(25,-8,18,22,0,0,Math.PI*2);ctx.fill();
  ctx.fillStyle='#111';ctx.fillRect(-23,-7,5,12);ctx.fillRect(26,-7,5,12);ctx.restore();
 }else if(enemy.kind==='guardian'){
  ctx.fillStyle='#817b91';ctx.fillRect(335,80,130,120);
  ctx.fillStyle='#fff';ctx.fillRect(355,105,25,15);ctx.fillRect(420,105,25,15);
  ctx.fillStyle='#ffda72';ctx.fillRect(363,108,10,9);ctx.fillRect(428,108,10,9);
  ctx.fillStyle='#393244';ctx.fillRect(365,155,70,9);
  ctx.fillStyle='#a9a2b7';ctx.fillRect(315,110,22,80);ctx.fillRect(463,110,22,80);
 }else if(enemy.kind==='king'){
  ctx.fillStyle=enemy.color;ctx.fillRect(355,95,90,100);
  ctx.fillStyle='#fff';ctx.fillRect(370,115,18,23);ctx.fillRect(412,115,18,23);
  ctx.fillStyle='#111';ctx.fillRect(378,121,6,12);ctx.fillRect(420,121,6,12);
  ctx.fillStyle='#d9b98b';ctx.fillRect(345,85,110,12);
  ctx.fillStyle='#fff';ctx.fillRect(380,160,40,5);
 }else{
  ctx.fillStyle=enemy.color;ctx.fillRect(355,95,90,100);
  ctx.fillStyle='#e7e2f5';ctx.fillRect(370,115,18,23);ctx.fillRect(412,115,18,23);
  ctx.fillStyle='#e33';ctx.fillRect(378,121,6,12);ctx.fillRect(420,121,6,12);
  ctx.fillStyle='#4d263e';ctx.fillRect(380,160,40,5);
 }
}
function drawScene(){
 ctx.fillStyle='#000';ctx.fillRect(0,0,W,H);
 for(let i=0;i<48;i++){
  ctx.fillStyle=i%3?'#25202b':'#42354b';
  ctx.fillRect((i*137)%W,(i*71)%H,3,3);
 }
 if(scene==='prologue'||scene==='flower'){
  ctx.fillStyle='#24152e';ctx.fillRect(0,260,W,170);
  for(let i=0;i<8;i++){
   ctx.fillStyle='#b7a04e';ctx.beginPath();
   ctx.ellipse(75+i*95,280+(i%2)*13,17,6,i*.3,0,Math.PI*2);ctx.fill();
  }
  human(400,240);
  if(scene==='flower'){
   ctx.fillStyle='#55bb55';ctx.beginPath();ctx.arc(530,205,40,0,Math.PI*2);ctx.fill();
   ctx.fillStyle='#fff';ctx.fillRect(515,190,7,9);ctx.fillRect(537,190,7,9);
  }
 }else if(['rootHall','ruins','afterRuins','miniboss','echo','mushroom','archives','train','trainBattle'].includes(scene)){
  ctx.fillStyle=scene==='mushroom'?'#152b2c':scene==='train'||scene==='trainBattle'?'#20232c':'#25202d';
  ctx.fillRect(0,240,W,190);
  for(let i=0;i<9;i++){
   ctx.fillStyle=scene==='mushroom'?'#285050':'#493a53';
   ctx.fillRect(i*100,180-(i%3)*15,80,100);
  }
  if(scene==='mushroom'){
   for(let i=0;i<7;i++){
    ctx.fillStyle=i%2?'#a45bd2':'#4bcac0';
    ctx.beginPath();ctx.arc(60+i*110,260-(i%3)*30,15,Math.PI,0);ctx.fill();
    ctx.fillRect(55+i*110,260-(i%3)*30,10,24);
   }
  }
  if(scene==='train'||scene==='trainBattle'){
   ctx.fillStyle='#3d414b';ctx.fillRect(40,90,720,170);
   ctx.strokeStyle='#aaa';ctx.lineWidth=4;
   ctx.strokeRect(60,105,145,95);ctx.strokeRect(230,105,145,95);
   ctx.strokeRect(400,105,145,95);ctx.strokeRect(570,105,145,95);
   if(scene==='trainBattle')txt('!!',400,70,35,'#ff4242');
  }
  if(scene==='echo'){txt('☠',485,200,65,'#fff');txt('☠',560,220,45,'#fff')}
  if(scene==='miniboss')txt('✦',400,140,70,'#ffd86c');
 }else if(scene==='city'){
  ctx.fillStyle='#15233b';ctx.fillRect(0,180,W,250);
  for(let i=0;i<8;i++){
   ctx.fillStyle='#293c60';ctx.fillRect(i*105,130+(i%2)*30,85,170);
   ctx.fillStyle='#9ee6ff';ctx.fillRect(i*105+20,155+(i%2)*30,10,18);
   ctx.fillRect(i*105+50,190+(i%2)*30,10,18);
  }
 }else if(scene==='castle'||scene==='epilogue'){
  ctx.fillStyle='#28212b';ctx.fillRect(0,160,W,270);
  ctx.fillStyle='#77623e';ctx.fillRect(90,90,620,280);
  ctx.fillStyle='#17131b';ctx.fillRect(310,150,180,220);
  txt('♛',400,215,60,'#ffd66b');human(250,280);
  if(scene==='epilogue'){
   ctx.fillStyle='#ffdf7e';ctx.globalAlpha=.5+Math.sin(frame*.08)*.3;
   ctx.fillRect(0,0,W,H);ctx.globalAlpha=1;
  }
 }
 if(state==='ending')txt('THE END',400,365,34,'#ffdf7e');
}
function draw(){
 if(['story','ending'].includes(state)){drawScene();return}
 ctx.fillStyle='#000';ctx.fillRect(0,0,W,H);
 for(let i=0;i<36;i++){
  ctx.fillStyle='#201c28';ctx.fillRect((i*97)%W,(i*53)%H,3,3);
 }
 monsterArt();txt(enemy.name,400,218,19);
 ctx.strokeStyle='#fff';ctx.lineWidth=3;ctx.strokeRect(arena.x,arena.y,arena.w,arena.h);

 if(state==='attack'){
  for(const b of bullets){
   ctx.fillStyle=b.color||'#fff';ctx.fillRect(b.x,b.y,b.s,b.s);
  }
  if(invuln<=0||Math.floor(frame/3)%2===0)heart(soul.x,soul.y,.8);
  txt('DODGE!',400,235,14,'#bbb');
 }else if(state==='fight'){
  ctx.strokeRect(250,310,300,22);
  ctx.fillStyle='#ffbd35';ctx.fillRect(255+meter*2.85,313,6,16);
  txt('SPACE: STRIKE',400,365,14);
 }else if(state==='win'){
  txt('VICTORY',400,300,32,'#ffdf7e');
 }else if(state==='dead'){
  ctx.fillStyle='#000';ctx.fillRect(0,0,W,H);
  heart(400,170,2);txt('GAME OVER',400,245,36,'#ff3038');
  txt('Press R to restart',400,285,20);
 }
}
function updateButtons(){
 const story=state==='story';
 $('fight').textContent=story?'CONTINUE':allowedAction==='fight'?'⚔ FIGHT (CHOSEN)':'⚔ FIGHT (LOCKED)';
 $('act').textContent=story?'CONTINUE':allowedAction==='act'?'☼ ACT (CHOSEN)':'☼ ACT (LOCKED)';
 $('item').textContent='♥ ITEM ('+inventory.length+')';
 $('mercy').textContent=state==='win'?'CONTINUE':story?'CONTINUE':'♡ MERCY';
 $('fight').disabled=story?false:state!=='menu'||allowedAction!=='fight';
 $('act').disabled=story?false:state!=='menu'||allowedAction!=='act';
 $('item').disabled=story?false:state!=='menu'||inventory.length===0;
 $('mercy').disabled=story?false:!(state==='menu'||state==='win');
}
function setStory(key){
 scene=key;state='story';line=0;
 $('chapter').textContent='SOULBOUND | '+key.toUpperCase();
 say(chapters[key][0]);updateButtons();
}
function nextStory(){
 if(state!=='story')return;
 line++;
 if(line>=chapters[scene].length){storyDone();return}
 say(chapters[scene][line]);
}
function storyDone(){
 if(scene==='prologue'){setStory('flower');return}
 if(scene==='flower'){startBattle('SMILING FLOWER','flower',22,'#e7bf35');return}
 if(scene==='rootHall'){setStory('ruins');return}
 if(scene==='ruins'){startBattle('SHY CAVE CRITTER','critter',26,'#76d58a');return}
 if(scene==='afterRuins'){setStory('echo');return}
 if(scene==='echo'){flags.metPip=true;setStory('mushroom');return}
 if(scene==='mushroom'){startBattle('TUNNEL STALKER','stalker',30,'#8d7ec6');return}
 if(scene==='miniboss'){startBattle('ANCIENT GUARDIAN','guardian',42,'#817b91');return}
 if(scene==='city'){setStory('archives');return}
 if(scene==='archives'){flags.key=true;setStory('train');return}
 if(scene==='train'){startBattle('INFECTED PASSENGER','infected',38,'#7b8d78');return}
 if(scene==='trainBattle'){setStory('castle');return}
 if(scene==='castle'){startBattle('KING AURON','king',72,'#c2a26b');return}
 if(scene==='epilogue'){
  state='ending';ending='PEACE';
  say('TRUE PEACE ENDING\nYou and the monsters step into a new world together.\n\nPress R to begin again.');
  updateButtons();
 }
}
function startBattle(name,kind,amount,color){
 enemy={name,kind,color};enemyHp=enemyMax=amount;mood=0;turn=0;
 state='menu';scene='battle';
 $('chapter').textContent='SOULBOUND | ENCOUNTER';
 chooseAction();
 say(name+' blocks your path.\nThe next action is chosen by fate...');
 updateButtons();
}
function chooseAction(){
 allowedAction=Math.random()<.5?'fight':'act';
 $('fight').textContent=allowedAction==='fight'?'⚔ FIGHT (CHOSEN)':'⚔ FIGHT (LOCKED)';
 $('act').textContent=allowedAction==='act'?'☼ ACT (CHOSEN)':'☼ ACT (LOCKED)';
}
function startAttack(){
 state='attack';bullets=[];soul={x:400,y:320};invuln=0;frame=0;
 const pattern=turn%4,count=10+Math.min(turn,6);
 for(let i=0;i<count;i++){
  if(pattern===0){
   bullets.push({x:rand(arena.x+10,arena.x+arena.w-10),y:arena.y-15-i*23,s:10,vx:rand(-.5,.5),vy:rand(2.2,3.6)});
  }else if(pattern===1){
   bullets.push({x:arena.x-18-i*31,y:rand(arena.y+10,arena.y+arena.h-10),s:10,vx:rand(2.3,3.6),vy:rand(-.4,.4)});
  }else if(pattern===2){
   bullets.push({x:arena.x+rand(0,arena.w),y:arena.y-15-i*24,s:9,vx:0,vy:rand(2.8,4),color:'#ffbd35'});
  }else{
   bullets.push({x:arena.x+rand(0,arena.w),y:arena.y+rand(0,arena.h),s:8,vx:rand(-1.5,1.5),vy:rand(1.4,2.8),color:'#c6a3ff'});
  }
 }
 say('ENEMY TURN! Dodge with WASD or ARROW KEYS.');
 updateButtons();
}
function finishAttack(){
 bullets=[];turn++;state='menu';chooseAction();
 say('You survived! Fate has chosen your next action: '+allowedAction.toUpperCase()+'.');
 updateButtons();
}
function doFight(){
 if(state!=='menu'||allowedAction!=='fight')return;
 state='fight';meter=0;meterDir=1;
 say('FIGHT was randomly selected for you! Press SPACE when the meter is near the center.');
 updateButtons();
}
function strike(){
 if(state!=='fight')return;
 const damage=Math.max(2,Math.round(18-Math.abs(meter-50)*.3));
 enemyHp-=damage;
 if(enemyHp<=0){
  enemyHp=0;state='win';kills++;gold+=10;xp+=12;
  if(xp>=24){lv++;maxHp+=4;hp=Math.min(maxHp,hp+4);xp-=24}
  say('You deal '+damage+' damage. '+enemy.name+' is defeated. +10 GOLD.');
  hud();updateButtons();
 }else{
  state='me
<!DOCTYPE html>
<html lang="en">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1">
<title>SOULBOUND: Beneath the Mountain</title>
<style>
*{box-sizing:border-box}
body{margin:0;background:#09070d;color:#fff;font-family:Consolas,"Courier New",monospace;display:flex;justify-content:center;padding:16px}
#game{width:min(940px,100%);background:#000;border:4px solid #eee;padding:16px;box-shadow:0 0 30px #33223e}
#top{display:flex;justify-content:space-between;gap:10px;flex-wrap:wrap;font-size:14px;margin-bottom:12px}
canvas{width:100%;height:auto;display:block;border:2px solid #eee;background:#000;image-rendering:pixelated}
#dialogue{white-space:pre-wrap;line-height:1.5;min-height:105px;border:2px solid #eee;padding:12px;margin-top:12px}
#actions{display:grid;grid-template-columns:repeat(4,minmax(0,1fr));gap:8px;margin-top:10px}
button{font:bold 15px Consolas,monospace;padding:12px 4px;background:#000;color:#ffbd35;border:2px solid #ffbd35;cursor:pointer}
button:hover:not(:disabled){background:#ffbd35;color:#000}
button:disabled{opacity:.28;cursor:not-allowed}
#hint{font-size:12px;color:#aaa;line-height:1.5;margin-top:12px}
#hpbar{display:inline-block;width:90px;height:12px;border:1px solid white;background:#461217;vertical-align:middle}
#hpfill{display:block;height:100%;background:#ff3038;width:100%}
@media(max-width:560px){
 #game{padding:8px}
 #dialogue{font-size:13px}
 button{font-size:11px;padding:10px 2px}
 #top{font-size:11px}
}
</style>
</head>
<body>
<main id="game">
 <div id="top">
  <span id="chapter">SOULBOUND | PROLOGUE</span>
  <span>LV <b id="lv">1</b> · HP <b id="hp">24</b>/<span id="maxhp">24</span> <span id="hpbar"><span id="hpfill"></span></span></span>
  <span id="gold">GOLD 0</span>
 </div>
 <canvas id="screen" width="800" height="430"></canvas>
 <div id="dialogue" role="status" aria-live="polite"></div>
 <div id="actions">
  <button id="fight">⚔ FIGHT</button>
  <button id="act">☼ ACT</button>
  <button id="item">♥ ITEM</button>
  <button id="mercy">♡ MERCY</button>
 </div>
 <div id="hint">
  Story: press ENTER or CONTINUE.
  Battle rule: FIGHT or ACT is randomly selected for each player turn.
  The other action is locked.
  Use ITEM to heal and MERCY when available.
  Dodge with WASD or ARROW KEYS.
  Press SPACE during a FIGHT meter.
  Press R after an ending or game over.
 </div>
</main>

<script>
(()=>{
const canvas=document.querySelector('#screen');
const ctx=canvas.getContext('2d');
const $=id=>document.getElementById(id);
const W=800,H=430;
const arena={x:220,y:245,w:360,h:145};

let state='story',scene='prologue',line=0,frame=0,last=0,keys={};
let hp=24,maxHp=24,gold=0,lv=1,xp=0,kills=0,mercyCount=0;
let enemy={name:'',kind:'slime',color:'#76d58a'};
let enemyHp=0,enemyMax=0,mood=0,turn=0;
let allowedAction='fight';
let soul={x:400,y:320},bullets=[],invuln=0;
let meter=0,meterDir=1,ending='';
let inventory=['Starberry','Starberry'];
let flags={key:false,metPip:false,bridge:false};

const chapters={
prologue:[
'Long before your story began, humans and monsters lived beneath the same sky.',
'Monsters could shape magic from their SOULS. Humans feared what they did not understand.',
'A war shattered the peace. Humans won, and the monsters were driven into the mountain.',
'Seven mages sealed the mountain with a barrier. No monster could cross it.',
'Generations passed. The war became a legend, and the mountain became a warning.',
'“Climb the mountain, and you may never come home.”',
'You followed a trail of golden flowers to the summit. The ground cracked beneath your feet.',
'You fell through the darkness. Wind roared in your ears, then—silence.',
'You wake in a bed of golden flowers under a ceiling of black stone.',
'There is no ladder. No phone signal. Only a narrow passage leading deeper underground.',
'Press ENTER or CONTINUE to begin your journey.'
],
flower:[
'A flower sways in the still air. Its petals are bright, but its smile feels practiced.',
'FLOWER: “Oh, hey! A human! You look lost.”',
'FLOWER: “Down here, your SOUL is the most important thing you have.”',
'It draws a tiny red heart in the dust. “That little heart is you. Keep it safe.”',
'FLOWER: “Let me teach you the rules. Everyone gets a turn, and everyone gets hurt.”',
'For one instant, its eyes turn flat and cold.',
'FLOWER: “Let’s see how you handle a little danger.”'
],
rootHall:[
'The flower disappears through a crack in the stone before you can ask its name.',
'You enter ROOT HALL, a maze of old pillars and roots thick as tree trunks.',
'Small lanterns burn with blue flame. Someone has been keeping them lit.',
'On a wall, you find a faded mural: a human child standing beside a monster child.',
'Someone has scratched out the human’s face.',
'A wooden sign points toward the RUINS. Another points toward a locked side door.',
'The door has a keyhole shaped like a star. You make a note to look for a key.'
],
ruins:[
'The ruins smell of rain even though no rain has fallen here in centuries.',
'A nervous little monster blocks the bridge. Its body looks like a soft green cloud.',
'CRITTER: “I was told to guard this bridge, but I forgot what I was guarding it from.”',
'It glances at your hands. “You are not going to hurt me, are you?”',
'You can only perform the action randomly assigned at the start of each turn.'
],
afterRuins:[
'The critter steps aside, relieved. It leaves a few coins beside the path.',
'You cross the bridge into a tunnel lined with cracked mirrors.',
'In each reflection, you see yourself a little older—walking, running, standing before a throne.',
'At the tunnel’s end, a sign reads: “THE ECHO GALLERY. DO NOT BELIEVE EVERYTHING YOU HEAR.”',
'From somewhere ahead, you hear a skeleton laughing at his own joke.'
],
echo:[
'Two skeletons stand beneath a chandelier made from old spoons.',
'MAGNUS: “HUMAN! I, MAGNUS, HAVE FINALLY FOUND YOU!”',
'PIP: “you found a broom closet, actually. the human is over there.”',
'MAGNUS: “I WAS TESTING THE CLOSET.”',
'Pip turns to you. “Sorry. My brother has a lot of confidence and not a lot of map skills.”',
'Magnus challenges you to a dramatic duel, then trips over his own cape.',
'Pip quietly slips you a note: “The king is scared. Don’t let anyone tell you otherwise.”',
'They point you toward the underground city. Magnus insists he is escorting you, then walks backward into a wall.'
],
mushroom:[
'You enter the MUSHROOM GROVE, where glowing caps light a forest of stone.',
'Every mushroom hums a different note. Together they make a strange, beautiful song.',
'A tiny vendor named NIM sits beside a cart of bottled starlight.',
'NIM: “Travelers pay with gold, stories, or secrets. You look short on all three.”',
'Nim gives you a Starberry anyway. “Pay it forward, little wanderer.”',
'A deep rumble shakes dust from the ceiling. Something enormous moves beneath the grove.'
],
miniboss:[
'The tunnel opens into a circular chamber. A stone guardian rises from the floor.',
'GUARDIAN: “ONLY THOSE WHO KNOW MERCY MAY PASS.”',
'Its eyes glow like two old stars. It does not seem eager to fight.',
'The guardian lowers its head, waiting to see what you will do.'
],
city:[
'The underground city of LANTERNFALL stretches around a lake of glowing blue water.',
'Families gather under paper lanterns. A band plays on a bridge. A child chases a floating orb.',
'Some monsters stare at you with fear. Others offer a wave, as if they have been waiting for you.',
'An old shopkeeper hands you a Starberry. “The king has called for a human. He believes your SOUL can break the barrier.”',
'You hear another rumor: the barrier may also be broken by a choice freely given.',
'You cross the market square toward the royal lift.'
],
archives:[
'The royal lift stops at the ARCHIVES, a library carved into the mountain’s heart.',
'Books fill the shelves from floor to ceiling. Some are written in human script, others in monster runes.',
'You find the journal of Queen Elara, who died long before you were born.',
'JOURNAL: “A barrier can be broken by power. But power is not the same as freedom.”',
'JOURNAL: “If a human and monster SOUL choose to trust one another, the seal may open without a sacrifice.”',
'The final page has been torn out. A golden key is tucked into the binding.',
'You take the key. Somewhere below, a bell rings three times.'
],
train:[
'The lift opens onto an underground railway platform. A train waits in a cloud of steam.',
'Pip is there, breathless. “the castle bridge is broken. this train is the only way across the chasm.”',
'Inside, the seats are empty. A pale figure is slumped by the far door.',
'The figure raises one trembling hand. “Please... help me...”',
'You take the stranger’s hand. Their fingers are ice-cold.',
'STRANGER: “I thought I could make it...” Their body shudders. Dark veins spread beneath their skin.',
'Pip backs away. “that’s not a person anymore. get ready!”',
'The stranger collapses, then rises as a monster. The train doors slam shut.'
],
trainBattle:[
'The infected passenger lunges. More creatures claw their way out of the dark carriages.',
'You fight your way through the train while it rattles over the chasm.',
'At last, the brakes scream. The doors slide open onto the castle’s outer platform.',
'Pip helps you down. “I’ll get the others to safety. you need to reach the king.”',
'Across the tracks, the royal gates begin to open.'
],
castle:[
'The castle is silent. Its halls are lined with portraits of monsters who never saw the surface.',
'At the throne room door, you find a small wooden toy horse. Someone has repaired its broken leg.',
'KING AURON stands beside a window overlooking the barrier: a shimmering dome of pale light.',
'AURON: “I have watched my people grow old beneath a sky they cannot touch.”',
'AURON: “Your SOUL could break the barrier. I will not pretend the price is small.”',
'AURON: “But I have promised them a future. I do not know how to keep that promise any other way.”',
'He removes his crown and places it on the throne.',
'AURON: “Show me what kind of human you are.”'
],
epilogue:[
'The battle ends. The king looks at you—not as a weapon, but as a child who fell into a terrible story.',
'Pip, Magnus, Nim, and the monsters of Lanternfall gather in the throne room.',
'The archive journal glows. Its missing final page appears in a shower of golden dust.',
'The page describes a third path: two SOULS, freely offered, can change the barrier instead of destroying it.',
'You reach out your hand. The king takes it.',
'Light flows across the mountain. The barrier opens like a curtain at dawn.',
'For the first time in centuries, monsters see the open sky.',
'Some cry. Some laugh. Magnus declares that he is “definitely not crying,” while Pip pats his shoulder.',
'Your journey is not over—but now, the world above is waiting.',
'THE END — Press R to play again.'
]
};

function say(t){$('dialogue').textContent=t}
function hud(){
 $('hp').textContent=hp;
 $('maxhp').textContent=maxHp;
 $('hpfill').style.width=(hp/maxHp*100)+'%';
 $('gold').textContent='GOLD '+gold;
 $('lv').textContent=lv;
}
function rand(a,b){return a+Math.random()*(b-a)}
function txt(t,x,y,size=20,color='#fff',align='center'){
 ctx.fillStyle=color;
 ctx.font=`bold ${size}px Consolas,monospace`;
 ctx.textAlign=align;
 ctx.fillText(t,x,y);
}
function heart(x,y,s=1,color='#ff3038'){
 ctx.save();ctx.translate(x,y);ctx.scale(s,s);ctx.fillStyle=color;
 ctx.beginPath();ctx.moveTo(0,7);
 ctx.bezierCurveTo(-19,-5,-11,-17,0,-8);
 ctx.bezierCurveTo(11,-17,19,-5,0,7);
 ctx.fill();ctx.restore();
}
function human(x,y,c='#c7a2ff'){
 ctx.fillStyle=c;ctx.fillRect(x-10,y-17,20,24);
 ctx.fillStyle='#f0c8a8';ctx.fillRect(x-8,y-31,16,15);
 ctx.fillStyle='#38284b';ctx.fillRect(x-9,y-34,18,6);
 ctx.fillStyle=c;ctx.fillRect(x-9,y+7,7,15);ctx.fillRect(x+2,y+7,7,15);
}
function monsterArt(){
 let bob=Math.sin(frame*.06)*4;
 if(enemy.kind==='flower'){
  ctx.fillStyle='#318d48';ctx.fillRect(390,120,20,70);
  for(let i=0;i<8;i++){
   ctx.fillStyle=i%2?'#f4d44b':'#e64c65';
   ctx.beginPath();
   ctx.ellipse(400+Math.cos(i*Math.PI/4)*30,110+Math.sin(i*Math.PI/4)*30,18,12,i*Math.PI/4,0,Math.PI*2);
   ctx.fill();
  }
  ctx.fillStyle='#e6bd38';ctx.beginPath();ctx.arc(400,110,22,0,Math.PI*2);ctx.fill();
  ctx.fillStyle='#111';ctx.fillRect(392,105,5,7);ctx.fillRect(404,105,5,7);ctx.fillRect(392,119,16,3);
 }else if(enemy.kind==='slime'||enemy.kind==='critter'){
  ctx.save();ctx.translate(400,145+bob);ctx.fillStyle=enemy.color;ctx.strokeStyle='#fff';ctx.lineWidth=3;
  ctx.beginPath();ctx.ellipse(0,0,78,56,0,0,Math.PI*2);ctx.fill();ctx.stroke();
  ctx.fillStyle='#fff';ctx.beginPath();
  ctx.ellipse(-25,-8,18,22,0,0,Math.PI*2);ctx.ellipse(25,-8,18,22,0,0,Math.PI*2);ctx.fill();
  ctx.fillStyle='#111';ctx.fillRect(-23,-7,5,12);ctx.fillRect(26,-7,5,12);ctx.restore();
 }else if(enemy.kind==='guardian'){
  ctx.fillStyle='#817b91';ctx.fillRect(335,80,130,120);
  ctx.fillStyle='#fff';ctx.fillRect(355,105,25,15);ctx.fillRect(420,105,25,15);
  ctx.fillStyle='#ffda72';ctx.fillRect(363,108,10,9);ctx.fillRect(428,108,10,9);
  ctx.fillStyle='#393244';ctx.fillRect(365,155,70,9);
  ctx.fillStyle='#a9a2b7';ctx.fillRect(315,110,22,80);ctx.fillRect(463,110,22,80);
 }else if(enemy.kind==='king'){
  ctx.fillStyle=enemy.color;ctx.fillRect(355,95,90,100);
  ctx.fillStyle='#fff';ctx.fillRect(370,115,18,23);ctx.fillRect(412,115,18,23);
  ctx.fillStyle='#111';ctx.fillRect(378,121,6,12);ctx.fillRect(420,121,6,12);
  ctx.fillStyle='#d9b98b';ctx.fillRect(345,85,110,12);
  ctx.fillStyle='#fff';ctx.fillRect(380,160,40,5);
 }else{
  ctx.fillStyle=enemy.color;ctx.fillRect(355,95,90,100);
  ctx.fillStyle='#e7e2f5';ctx.fillRect(370,115,18,23);ctx.fillRect(412,115,18,23);
  ctx.fillStyle='#e33';ctx.fillRect(378,121,6,12);ctx.fillRect(420,121,6,12);
  ctx.fillStyle='#4d263e';ctx.fillRect(380,160,40,5);
 }
}
function drawScene(){
 ctx.fillStyle='#000';ctx.fillRect(0,0,W,H);
 for(let i=0;i<48;i++){
  ctx.fillStyle=i%3?'#25202b':'#42354b';
  ctx.fillRect((i*137)%W,(i*71)%H,3,3);
 }
 if(scene==='prologue'||scene==='flower'){
  ctx.fillStyle='#24152e';ctx.fillRect(0,260,W,170);
  for(let i=0;i<8;i++){
   ctx.fillStyle='#b7a04e';ctx.beginPath();
   ctx.ellipse(75+i*95,280+(i%2)*13,17,6,i*.3,0,Math.PI*2);ctx.fill();
  }
  human(400,240);
  if(scene==='flower'){
   ctx.fillStyle='#55bb55';ctx.beginPath();ctx.arc(530,205,40,0,Math.PI*2);ctx.fill();
   ctx.fillStyle='#fff';ctx.fillRect(515,190,7,9);ctx.fillRect(537,190,7,9);
  }
 }else if(['rootHall','ruins','afterRuins','miniboss','echo','mushroom','archives','train','trainBattle'].includes(scene)){
  ctx.fillStyle=scene==='mushroom'?'#152b2c':scene==='train'||scene==='trainBattle'?'#20232c':'#25202d';
  ctx.fillRect(0,240,W,190);
  for(let i=0;i<9;i++){
   ctx.fillStyle=scene==='mushroom'?'#285050':'#493a53';
   ctx.fillRect(i*100,180-(i%3)*15,80,100);
  }
  if(scene==='mushroom'){
   for(let i=0;i<7;i++){
    ctx.fillStyle=i%2?'#a45bd2':'#4bcac0';
    ctx.beginPath();ctx.arc(60+i*110,260-(i%3)*30,15,Math.PI,0);ctx.fill();
    ctx.fillRect(55+i*110,260-(i%3)*30,10,24);
   }
  }
  if(scene==='train'||scene==='trainBattle'){
   ctx.fillStyle='#3d414b';ctx.fillRect(40,90,720,170);
   ctx.strokeStyle='#aaa';ctx.lineWidth=4;
   ctx.strokeRect(60,105,145,95);ctx.strokeRect(230,105,145,95);
   ctx.strokeRect(400,105,145,95);ctx.strokeRect(570,105,145,95);
   if(scene==='trainBattle')txt('!!',400,70,35,'#ff4242');
  }
  if(scene==='echo'){txt('☠',485,200,65,'#fff');txt('☠',560,220,45,'#fff')}
  if(scene==='miniboss')txt('✦',400,140,70,'#ffd86c');
 }else if(scene==='city'){
  ctx.fillStyle='#15233b';ctx.fillRect(0,180,W,250);
  for(let i=0;i<8;i++){
   ctx.fillStyle='#293c60';ctx.fillRect(i*105,130+(i%2)*30,85,170);
   ctx.fillStyle='#9ee6ff';ctx.fillRect(i*105+20,155+(i%2)*30,10,18);
   ctx.fillRect(i*105+50,190+(i%2)*30,10,18);
  }
 }else if(scene==='castle'||scene==='epilogue'){
  ctx.fillStyle='#28212b';ctx.fillRect(0,160,W,270);
  ctx.fillStyle='#77623e';ctx.fillRect(90,90,620,280);
  ctx.fillStyle='#17131b';ctx.fillRect(310,150,180,220);
  txt('♛',400,215,60,'#ffd66b');human(250,280);
  if(scene==='epilogue'){
   ctx.fillStyle='#ffdf7e';ctx.globalAlpha=.5+Math.sin(frame*.08)*.3;
   ctx.fillRect(0,0,W,H);ctx.globalAlpha=1;
  }
 }
 if(state==='ending')txt('THE END',400,365,34,'#ffdf7e');
}
function draw(){
 if(['story','ending'].includes(state)){drawScene();return}
 ctx.fillStyle='#000';ctx.fillRect(0,0,W,H);
 for(let i=0;i<36;i++){
  ctx.fillStyle='#201c28';ctx.fillRect((i*97)%W,(i*53)%H,3,3);
 }
 monsterArt();txt(enemy.name,400,218,19);
 ctx.strokeStyle='#fff';ctx.lineWidth=3;ctx.strokeRect(arena.x,arena.y,arena.w,arena.h);

 if(state==='attack'){
  for(const b of bullets){
   ctx.fillStyle=b.color||'#fff';ctx.fillRect(b.x,b.y,b.s,b.s);
  }
  if(invuln<=0||Math.floor(frame/3)%2===0)heart(soul.x,soul.y,.8);
  txt('DODGE!',400,235,14,'#bbb');
 }else if(state==='fight'){
  ctx.strokeRect(250,310,300,22);
  ctx.fillStyle='#ffbd35';ctx.fillRect(255+meter*2.85,313,6,16);
  txt('SPACE: STRIKE',400,365,14);
 }else if(state==='win'){
  txt('VICTORY',400,300,32,'#ffdf7e');
 }else if(state==='dead'){
  ctx.fillStyle='#000';ctx.fillRect(0,0,W,H);
  heart(400,170,2);txt('GAME OVER',400,245,36,'#ff3038');
  txt('Press R to restart',400,285,20);
 }
}
function updateButtons(){
 const story=state==='story';
 $('fight').textContent=story?'CONTINUE':allowedAction==='fight'?'⚔ FIGHT (CHOSEN)':'⚔ FIGHT (LOCKED)';
 $('act').textContent=story?'CONTINUE':allowedAction==='act'?'☼ ACT (CHOSEN)':'☼ ACT (LOCKED)';
 $('item').textContent='♥ ITEM ('+inventory.length+')';
 $('mercy').textContent=state==='win'?'CONTINUE':story?'CONTINUE':'♡ MERCY';
 $('fight').disabled=story?false:state!=='menu'||allowedAction!=='fight';
 $('act').disabled=story?false:state!=='menu'||allowedAction!=='act';
 $('item').disabled=story?false:state!=='menu'||inventory.length===0;
 $('mercy').disabled=story?false:!(state==='menu'||state==='win');
}
function setStory(key){
 scene=key;state='story';line=0;
 $('chapter').textContent='SOULBOUND | '+key.toUpperCase();
 say(chapters[key][0]);updateButtons();
}
function nextStory(){
 if(state!=='story')return;
 line++;
 if(line>=chapters[scene].length){storyDone();return}
 say(chapters[scene][line]);
}
function storyDone(){
 if(scene==='prologue'){setStory('flower');return}
 if(scene==='flower'){startBattle('SMILING FLOWER','flower',22,'#e7bf35');return}
 if(scene==='rootHall'){setStory('ruins');return}
 if(scene==='ruins'){startBattle('SHY CAVE CRITTER','critter',26,'#76d58a');return}
 if(scene==='afterRuins'){setStory('echo');return}
 if(scene==='echo'){flags.metPip=true;setStory('mushroom');return}
 if(scene==='mushroom'){startBattle('TUNNEL STALKER','stalker',30,'#8d7ec6');return}
 if(scene==='miniboss'){startBattle('ANCIENT GUARDIAN','guardian',42,'#817b91');return}
 if(scene==='city'){setStory('archives');return}
 if(scene==='archives'){flags.key=true;setStory('train');return}
 if(scene==='train'){startBattle('INFECTED PASSENGER','infected',38,'#7b8d78');return}
 if(scene==='trainBattle'){setStory('castle');return}
 if(scene==='castle'){startBattle('KING AURON','king',72,'#c2a26b');return}
 if(scene==='epilogue'){
  state='ending';ending='PEACE';
  say('TRUE PEACE ENDING\nYou and the monsters step into a new world together.\n\nPress R to begin again.');
  updateButtons();
 }
}
function startBattle(name,kind,amount,color){
 enemy={name,kind,color};enemyHp=enemyMax=amount;mood=0;turn=0;
 state='menu';scene='battle';
 $('chapter').textContent='SOULBOUND | ENCOUNTER';
 chooseAction();
 say(name+' blocks your path.\nThe next action is chosen by fate...');
 updateButtons();
}
function chooseAction(){
 allowedAction=Math.random()<.5?'fight':'act';
 $('fight').textContent=allowedAction==='fight'?'⚔ FIGHT (CHOSEN)':'⚔ FIGHT (LOCKED)';
 $('act').textContent=allowedAction==='act'?'☼ ACT (CHOSEN)':'☼ ACT (LOCKED)';
}
function startAttack(){
 state='attack';bullets=[];soul={x:400,y:320};invuln=0;frame=0;
 const pattern=turn%4,count=10+Math.min(turn,6);
 for(let i=0;i<count;i++){
  if(pattern===0){
   bullets.push({x:rand(arena.x+10,arena.x+arena.w-10),y:arena.y-15-i*23,s:10,vx:rand(-.5,.5),vy:rand(2.2,3.6)});
  }else if(pattern===1){
   bullets.push({x:arena.x-18-i*31,y:rand(arena.y+10,arena.y+arena.h-10),s:10,vx:rand(2.3,3.6),vy:rand(-.4,.4)});
  }else if(pattern===2){
   bullets.push({x:arena.x+rand(0,arena.w),y:arena.y-15-i*24,s:9,vx:0,vy:rand(2.8,4),color:'#ffbd35'});
  }else{
   bullets.push({x:arena.x+rand(0,arena.w),y:arena.y+rand(0,arena.h),s:8,vx:rand(-1.5,1.5),vy:rand(1.4,2.8),color:'#c6a3ff'});
  }
 }
 say('ENEMY TURN! Dodge with WASD or ARROW KEYS.');
 updateButtons();
}
function finishAttack(){
 bullets=[];turn++;state='menu';chooseAction();
 say('You survived! Fate has chosen your next action: '+allowedAction.toUpperCase()+'.');
 updateButtons();
}
function doFight(){
 if(state!=='menu'||allowedAction!=='fight')return;
 state='fight';meter=0;meterDir=1;
 say('FIGHT was randomly selected for you! Press SPACE when the meter is near the center.');
 updateButtons();
}
function strike(){
 if(state!=='fight')return;
 const damage=Math.max(2,Math.round(18-Math.abs(meter-50)*.3));
 enemyHp-=damage;
 if(enemyHp<=0){
  enemyHp=0;state='win';kills++;gold+=10;xp+=12;
  if(xp>=24){lv++;maxHp+=4;hp=Math.min(maxHp,hp+4);xp-=24}
  say('You deal '+damage+' damage. '+enemy.name+' is defeated. +10 GOLD.');
  hud();updateButtons();
 }else{
  state='menu';
  say('You deal '+damage+' damage. '+enemy.name+' has '+enemyHp+' HP left.');
  startAttack();
 }
}
function doAct(){
 if(state!=='menu'||allowedAction!=='act')return;
 mood++;
 let msg='';
 if(enemy.kind==='flower'){
  msg='You ask why the flower is smiling. “Because somebody has to make this fun,” it says, but its voice trembles.';
 }else if(enemy.kind==='king'){
  msg='You ask the king what he would do if there were another way. He looks toward the window. “I would choose it.”';
 }else if(enemy.kind==='guardian'){
  msg='You bow your head and acknowledge the guardian’s duty. Its ancient eyes glow more softly.';
 }else if(enemy.kind==='infected'){
  msg='You call the passenger by the name written on their ticket. For one second, they seem to hear you.';
 }else{
  msg='You speak gently to '+enemy.name+'. Its posture relaxes, just a little.';
 }
 if(mood>=2&&enemy.kind!=='flower')msg+='\nIt seems ready to be spared.';
 say('ACT was randomly selected for you!\n'+msg);
 startAttack();
}
function useItem(){
 if(state==='story'){nextStory();return}
 if(state!=='menu'||!inventory.length)return;
 if(hp>=maxHp){say('Your HP is already full. The Starberry is saved.');return}
 inventory.pop();hp=Math.min(maxHp,hp+10);hud();
 say('You eat a Starberry and restore 10 HP.');
 startAttack();
}
function continueAfterWin(){
 if(enemy.kind==='flower'){setStory('rootHall');return}
 if(enemy.kind==='critter'){gold+=6;setStory('afterRuins');return}
 if(enemy.kind==='stalker'){setStory('miniboss');return}
 if(enemy.kind==='guardian'){gold+=15;flags.bridge=true;setStory('city');return}
 if(enemy.kind==='infected'){setStory('trainBattle');return}
 if(enemy.kind==='king'){setStory('epilogue');return}
}
function mercy(){
 if(state==='story'){nextStory();return}
 if(state==='win'){
  if(enemy.kind==='king'&&mood>=2){setStory('epilogue');return}
  continueAfterWin();hud();return;
 }
 if(state!=='menu')return;
 if(enemy.kind!=='flower'&&mood>=2){
  mercyCount++;gold+=5;state='win';
  say('You spare '+enemy.name+'. It steps aside and leaves 5 GOLD.');
  hud();updateButtons();
 }else{
  say(enemy.kind==='flower'
   ?'The flower laughs. “You cannot spare something that never meant you well.”'
   :'Not ready to be spared yet. Fate will choose your next action after the enemy turn.');
  startAttack();
 }
}
function reset(){
 state='story';scene='prologue';line=0;frame=0;
 hp=maxHp=24;gold=0;lv=1;xp=0;kills=0;mercyCount=0;
 inventory=['Starberry','Starberry'];
 flags={key:false,metPip:false,bridge:false};
 ending='';hud();setStory('prologue');
}
$('fight').addEventListener('click',()=>{
 if(state==='story'){nextStory();return}
 if(state==='fight'){strike();return}
 if(state==='menu'&&allowedAction==='fight')doFight();
});
$('act').addEventListener('click',()=>{
 if(state==='story'){nextStory();return}
 if(state==='menu'&&allowedAction==='act')doAct();
});
$('item').addEventListener('click',useItem);
$('mercy').addEventListener('click',mercy);
window.addEventListener('keydown',e=>{
 const k=e.key.toLowerCase();
 if(['arrowup','arrowdown','arrowleft','arrowright',' '].includes(k))e.preventDefault();
 keys[k]=true;
 if(k===' '&&state==='fight')strike();
 if(k==='enter'&&state==='story')nextStory();
 if(k==='r'&&(state==='ending'||state==='dead'))reset();
 if(k==='escape'&&state==='attack')finishAttack();
});
window.addEventListener('keyup',e=>keys[e.key.toLowerCase()]=false);
function loop(ts){
 const dt=Math.min(2,(ts-last)/16.67||1);last=ts;frame++;
 if(state==='fight'){
  meter+=meterDir*2.8*dt;
  if(meter>=100){meter=100;meterDir=-1}
  if(meter<=0){meter=0;meterDir=1}
 }
 if(state==='attack'){
  const speed=3.7*dt;
  if(keys.arrowleft||keys.a)soul.x-=speed*1.7;
  if(keys.arrowright||keys.d)soul.x+=speed*1.7;
  if(keys.arrowup||keys.w)soul.y-=speed*1.7;
  if(keys.arrowdown||keys.s)soul.y+=speed*1.7;
  soul.x=Math.max(arena.x+12,Math.min(arena.x+arena.w-12,soul.x));
  soul.y=Math.max(arena.y+12,Math.min(arena.y+arena.h-12,soul.y));
  if(invuln>0)invuln-=dt;
  for(const b of bullets){
   b.x+=b.vx*dt;b.y+=b.vy*dt;
   if(invuln<=0&&Math.abs(soul.x-(b.x+b.s/2))<13&&Math.abs(soul.y-(b.y+b.s/2))<13){
    hp=Math.max(0,hp-2);invuln=35;hud();
    if(hp<=0){
     state='dead';say('Your SOUL has fallen...\nPress R to restart.');
     updateButtons();break;
    }
   }
  }
  bullets=bullets.filter(b=>b.x>-30&&b.x<W+30&&b.y>-30&&b.y<H+30);
  if(frame%72===0&&bullets.length<22){
   bullets.push({x:rand(arena.x+10,arena.x+arena.w-10),y:arena.y-10,s:10,vx:rand(-.7,.7),vy:rand(2.7,4)});
  }
  if(state==='attack'&&frame>280)finishAttack();
 }
 draw();requestAnimationFrame(loop);
}
hud();setStory('prologue');requestAnimationFrame(loop);
})();
</script>
</body>
</html>
nu';
  say('You deal '+damage+' damage. '+enemy.name+' has '+enemyHp+' HP left.');
  startAttack();
 }
}
function doAct(){
 if(state!=='menu'||allowedAction!=='act')return;
 mood++;
 let msg='';
 if(enemy.kind==='flower'){
  msg='You ask why the flower is smiling. “Because somebody has to make this fun,” it says, but its voice trembles.';
 }else if(enemy.kind==='king'){
  msg='You ask the king what he would do if there were another way. He looks toward the window. “I would choose it.”';
 }else if(enemy.kind==='guardian'){
  msg='You bow your head and acknowledge the guardian’s duty. Its ancient eyes glow more softly.';
 }else if(enemy.kind==='infected'){
  msg='You call the passenger by the name written on their ticket. For one second, they seem to hear you.';
 }else{
  msg='You speak gently to '+enemy.name+'. Its posture relaxes, just a little.';
 }
 if(mood>=2&&enemy.kind!=='flower')msg+='\nIt seems ready to be spared.';
 say('ACT was randomly selected for you!\n'+msg);
 startAttack();
}
function useItem(){
 if(state==='story'){nextStory();return}
 if(state!=='menu'||!inventory.length)return;
 if(hp>=maxHp){say('Your HP is already full. The Starberry is saved.');return}
 inventory.pop();hp=Math.min(maxHp,hp+10);hud();
 say('You eat a Starberry and restore 10 HP.');
 startAttack();
}
function continueAfterWin(){
 if(enemy.kind==='flower'){setStory('rootHall');return}
 if(enemy.kind==='critter'){gold+=6;setStory('afterRuins');return}
 if(enemy.kind==='stalker'){setStory('miniboss');return}
 if(enemy.kind==='guardian'){gold+=15;flags.bridge=true;setStory('city');return}
 if(enemy.kind==='infected'){setStory('trainBattle');return}
 if(enemy.kind==='king'){setStory('epilogue');return}
}
function mercy(){
 if(state==='story'){nextStory();return}
 if(state==='win'){
  if(enemy.kind==='king'&&mood>=2){setStory('epilogue');return}
  continueAfterWin();hud();return;
 }
 if(state!=='menu')return;
 if(enemy.kind!=='flower'&&mood>=2){
  mercyCount++;gold+=5;state='win';
  say('You spare '+enemy.name+'. It steps aside and leaves 5 GOLD.');
  hud();updateButtons();
 }else{
  say(enemy.kind==='flower'
   ?'The flower laughs. “You cannot spare something that never meant you well.”'
   :'Not ready to be spared yet. Fate will choose your next action after the enemy turn.');
  startAttack();
 }
}
function reset(){
 state='story';scene='prologue';line=0;frame=0;
 hp=maxHp=24;gold=0;lv=1;xp=0;kills=0;mercyCount=0;
 inventory=['Starberry','Starberry'];
 flags={key:false,metPip:false,bridge:false};
 ending='';hud();setStory('prologue');
}
$('fight').addEventListener('click',()=>{
 if(state==='story'){nextStory();return}
 if(state==='fight'){strike();return}
 if(state==='menu'&&allowedAction==='fight')doFight();
});
$('act').addEventListener('click',()=>{
 if(state==='story'){nextStory();return}
 if(state==='menu'&&allowedAction==='act')doAct();
});
$('item').addEventListener('click',useItem);
$('mercy').addEventListener('click',mercy);
window.addEventListener('keydown',e=>{
 const k=e.key.toLowerCase();
 if(['arrowup','arrowdown','arrowleft','arrowright',' '].includes(k))e.preventDefault();
 keys[k]=true;
 if(k===' '&&state==='fight')strike();
 if(k==='enter'&&state==='story')nextStory();
 if(k==='r'&&(state==='ending'||state==='dead'))reset();
 if(k==='escape'&&state==='attack')finishAttack();
});
window.addEventListener('keyup',e=>keys[e.key.toLowerCase()]=false);
function loop(ts){
 const dt=Math.min(2,(ts-last)/16.67||1);last=ts;frame++;
 if(state==='fight'){
  meter+=meterDir*2.8*dt;
  if(meter>=100){meter=100;meterDir=-1}
  if(meter<=0){meter=0;meterDir=1}
 }
 if(state==='attack'){
  const speed=3.7*dt;
  if(keys.arrowleft||keys.a)soul.x-=speed*1.7;
  if(keys.arrowright||keys.d)soul.x+=speed*1.7;
  if(keys.arrowup||keys.w)soul.y-=speed*1.7;
  if(keys.arrowdown||keys.s)soul.y+=speed*1.7;
  soul.x=Math.max(arena.x+12,Math.min(arena.x+arena.w-12,soul.x));
  soul.y=Math.max(arena.y+12,Math.min(arena.y+arena.h-12,soul.y));
  if(invuln>0)invuln-=dt;
  for(const b of bullets){
   b.x+=b.vx*dt;b.y+=b.vy*dt;
   if(invuln<=0&&Math.abs(soul.x-(b.x+b.s/2))<13&&Math.abs(soul.y-(b.y+b.s/2))<13){
    hp=Math.max(0,hp-2);invuln=35;hud();
    if(hp<=0){
     state='dead';say('Your SOUL has fallen...\nPress R to restart.');
     updateButtons();break;
    }
   }
  }
  bullets=bullets.filter(b=>b.x>-30&&b.x<W+30&&b.y>-30&&b.y<H+30);
  if(frame%72===0&&bullets.length<22){
   bullets.push({x:rand(arena.x+10,arena.x+arena.w-10),y:arena.y-10,s:10,vx:rand(-.7,.7),vy:rand(2.7,4)});
  }
  if(state==='attack'&&frame>280)finishAttack();
 }
 draw();requestAnimationFrame(loop);
}
hud();setStory('prologue');requestAnimationFrame(loop);
})();
</script>
</body>
</html>

