# Star-Wars-Tower-Defense
Ez egy bázisvédelmi stratégiai játék, ahol a cél a támadások kivédésével való pont szerzés. Fentről folyamatosan nehezedő ellenségek éreznek, akiket a saját karaktereink segítségével távol kell tartani a főhadiszállásunktó. A termelődő erőpontjainkkal helyezhetünk harcosokat a csatatérre. A játék addig tart amíg a bázisunknak van életereje.

Játékszabályok és mechanikák
Erőforrás-kezelés: Az egységek leidézéséhez Erőre (Force) van szükség. Ez az érték folyamatosan, automatikusan újratermelődik a háttérben a maximális kapacitásig.

Egységek lehelyezése: Minden egység egyedi Erő-költséggel, életerővel (HP), sebzéssel (Damage), támadási sebességgel, hatótávval (Range) és mozgási sebességgel rendelkezik.

Csata és mesterséges intelligencia: A lehelyezett egységek automatikusan elindulnak előre (felfelé), és ha ellenséget észlelnek a hatótávolságukon belül, harcba bocsátkoznak vele. Az ellenséges egységek fentről lefelé masíroznak a bázis irányába.

Folyamatosan nehezedő hullámok: A játék előrehaladtával a játékidő (Game Time) növekedésével az ellenséges egységek leidézésének gyakorisága (Spawn Interval) folyamatosan rövidül, így a hullámok egyre sűrűbbé és nehezebbé válnak.

Győzelem és vereség feltételei: A játék addig tart, amíg a játékos bázisának életereje 0-ra nem csökken. Győzelem klasszikus értelemben nincs (végtelenített túlélő mód), a cél a lehető legmagasabb pontszám (Score) és legtöbb elpusztított ellenség (Kills) elérése. Ha a bázis megsemmisül, a játékos a "Game Over" képernyőre kerül.

Irányítás
A játék teljes mértékben egérrel vagy érintőképernyővel (mobilbarát reszponzív skálázás) vezérelhető:

Egység kiválasztása: A képernyő alján található kártyák (Cards) egyikére kattintva kijelölhető a leidézni kívánt egység (ha van rá elég Erő).

Lehelyezés a pályára: A battlefield (csatatér) területére kattintva az egység azonnal létrejön a kattintás X koordinátáján, a játékos leidézési zónájának fix magasságában (Y koordináta).

2. Magas szintű logika (Architektúra és Kódfelépítés)
A játék egyetlen, tisztán kliensoldali objektumorientált JavaScript struktúrára épül, amely különálló osztályokat, egy globális állapotkezelőt és egy központi játékciklust használ.

Főbb globális objektumok
GS (Game State): A játék globális állapotát tároló objektum. Nyomon követi a pontszámot (score), az elpusztított ellenségek számát (kills), az aktuális Erőt (force), a játékidőt (gameTime), a játék végét (isGameOver), valamint tömbökben tárolja az összes aktív egységet (units) és lövedéket (projectiles).

UI (User Interface): A DOM elemek frissítéséért (Erő-csík, életerő, pontszámok, kártyák aktív/inaktív állapota) felelős komponens.

Főbb JavaScript osztályok
Unit (Egység osztály): Minden csatatéren lévő karakter (mind a szövetséges Köztársaságiak, mind az ellenséges Birodalmiak, mind maga a Főbázis) a Unit osztály egy-egy példánya.

Főbb tulajdonságai: id, side (republic/empire), type, hp, maxHp, damage, range, speed, x, y, és a hozzá tartozó DOM elem (el).

Főbb metódusai:

update(dt): Minden képkockánál lefut. Keresi a legközelebbi ellenséget a látótávolságon belül. Ha lőtávolon kívül van, mozgatja az egységet; ha lőtávolon belül van, megállítja és elindítja a támadási időzítőt (attackTimer).

attack(target): Sebzést okoz a célpontnak. Ha az egység távolsági harcos (pl. Clone Trooper vagy Stormtrooper), akkor nem közvetlenül sebez, hanem létrehoz egy új Projectile példányt.

takeDamage(amount): Csökkenti az életerőt, frissíti az egység feletti HP-csíkot, és ha a HP 0 alá esik, megsemmisíti az egységet.

Projectile (Lövedék osztály):
A távolsági egységek által kilőtt lézernyalábokat vagy plazmalövedékeket kezeli.

Főbb tulajdonságai: Kezdőpont (x, y), célpont egység (target), sebesség és a vizuális lézersugár megjelenítése.

Főbb metódusai: * update(dt): Lineárisan mozgatja a lövedéket a célpont aktuális pozíciója felé. Amikor eléri a célpontot, kiváltja a célpont takeDamage függvényét, majd a lövedéket törlésre jelöli (done = true).

Kulcsfontosságú függvények és Játékciklus
gameLoop(now): A játék szíve. A requestAnimationFrame segítségével folyamatosan fut másodpercenként 60-szor. Kiszámítja az eltelt időt (dt), regenerálja a játékos Erő-erőforrását, kezeli az ellenséges egységek automatikus időzített leidézését (spawnEnemy()), és meghívja az összes aktív egység és lövedék update() metódusát, végül letisztítja a halott egységeket és frissíti a kezelőfelületet (UI.update()).

spawnEnemy(): A játékidő függvényében generál véletlenszerűen ellenséges Birodalmi egységeket (pl. Stormtrooper, Shadow Trooper, vagy elit AT-ST / Darth Vader bossokat) a battlefield tetejére.

spawnUnit(type, x, y): Létrehozza a játékos által vásárolt egységet a megadott koordinátákon, levonja az Erő-költséget, és regisztrálja az egységet a GS.units tömbbe.

3. AI eszközök használata
A fejlesztési folyamat során több különböző generatív Mesterséges Intelligencia (AI) eszközt is igénybe vettünk, amelyek jelentősen felgyorsították a munkát, és lehetővé tették egy komplex, vizuálisan is vonzó prototípus gyors elkészítését:

Claude (Anthropic): A Claude-ot elsősorban a kód megírására és strukturálására használtuk. Kimagasló segítséget nyújtott a tiszta, egyetlen fájlba sűríthető (Single-file Component) HTML/CSS/JS architektúra kialakításában. Segített a matematikai alapú ütközésdetektálás, a lövedékek célra követési logikájának megírásában, valamint a kártyaalapú lehelyezési mechanizmus és a folyamatosan nehezedő játékciklus (Game Loop) stabil implementálásában.

ChatGPT (OpenAI) & Gemini (Google): Ezen eszközöket elsősorban a kreatív folyamatokhoz, kifejezetten a képek szerkesztéséhez és a vizuális elemek előkészítéséhez használtuk. Segítséget nyújtottak a Star Wars hangulathoz illeszkedő, sötét, lávás-vulkanikus (Mustafar stílusú) hátterek, textúrák és a karakterek kártyáinak grafikai optimalizálásában, valamint a base64 formátumú vizuális assetek kódba illeszthető előállításában. Ennek köszönhetően a játék külsőleg is hozza a sci-fi atmoszférát anélkül, hogy külső szerverekről kellene nagy méretű képeket letöltenie.
