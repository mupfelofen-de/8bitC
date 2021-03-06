---


---

<p><img src="8bitC.jpg" alt="8-bit C"></p>
<h1 id="c-portabile-e-ottimizzato-per-gli-8-bit">C portabile e ottimizzato per gli 8-bit</h1>
<p>Questo articolo descrive alcune tecniche per ottimizzare codice in ANSI C per <strong>tutti</strong> i sistemi 8-bit <em>vintage</em>, cioè computer, console, handheld, calcolatrici scientifiche dalla fine degli anni 70 fino a metà degli anni 90 ed in particolare sistemi basati sulle seguenti <em>architetture</em> (e architetture derivate e retrocompatibili):</p>
<ul>
<li>Intel 8080 (*)</li>
<li>MOS 6502</li>
<li>Motorola 6809</li>
<li>Zilog Z80 (*)</li>
</ul>
<p>(*) Lo Zilog Z80 è una estensione dell’Intel 8080, quindi un binario Intel 8080 sarà utilizzabile anche su un sistema con Z80 ma il viceversa non è vero.</p>
<p>Buona parte di queste tecniche sono valide su altre architetture 8-bit come quella del COSMAC 1802 e quella del microcontrollore Intel 8051.</p>
<p>Lo scopo di questo articolo è duplice:</p>
<ol>
<li>descrivere tecniche generali per <strong>ottimizzare</strong> il codice C su <strong>tutti</strong> i sistemi 8-bit</li>
<li>descrivere tecniche generiche per scrivere codice <strong>portabile</strong>, cioè valido e compilabile su <strong>tutti</strong> i sistemi 8-bit indipendentemente dal fatto che un sistema sia supportato esplicitamente da un compilatore o meno</li>
</ol>
<h2 id="premesse">Premesse</h2>
<p>Questo articolo <strong>non</strong> è un manuale introduttivo al linguaggio <em>C</em> e richiede</p>
<ul>
<li>conoscenza del linguaggio <em>C</em>;</li>
<li>conoscenza della programmazione strutturata e a oggetti;</li>
<li>conoscenza dell’uso di compilatori e linker.</li>
</ul>
<p>Inoltre questo articolo <strong>non</strong> si occuperà in profondità di alcuni argomenti avanzati quali:</p>
<ul>
<li>gli ambiti specifici della programmazione come grafica, suono, input/output, etc.</li>
<li>l’interazione tra C e Assembly.</li>
</ul>
<p>Questi argomenti avanzati sono molto importanti e meriterebbero degli articoli separati a loro dedicati.</p>
<h2 id="terminologia">Terminologia</h2>
<p>Introduciamo alcuni termini che saranno ricorrenti in questo articolo:</p>
<ul>
<li>Un <em>sistema</em> è un qualunque tipo di macchina dotata di processore come computer, console, handheld, calcolatrici, sistemi embedded, etc.</li>
<li>Un <em>target</em> di un compilatore è un sistema supportato dal compilatore, cioè un sistema per il quale il compilatore mette a disposizione supporto specifico come librerie e la generazione di un binario in formato specifico.</li>
<li>Un <em>architettura</em> è un tipo di processore (Intel 8080, 6502, Zilog Z80, Motorola 6809, etc.) . Un <em>target</em> appartiene quindi ad una sola architettura data dal suo processore (con rare eccezioni come il Commodore 128 che ha sia un processore derivato dal 6502 e uno Zilog Z80).</li>
</ul>
<h2 id="cross-compilatori-multi-target">Cross-compilatori multi-target</h2>
<p>Per produrre i nostri binari 8-bit consigliamo l’uso di <em>cross compilatori</em> <em>multi-target</em> (cioè compilatori eseguiti su PC che producono binari per diversi <em>target</em>).</p>
<h3 id="cross-compilatori-vs-compilatori-nativi">Cross-compilatori vs compilatori nativi</h3>
<p>Non consigliamo l’uso di compilatori <em>nativi</em> perché sarebbero molto scomodi (anche se usati all’interno di un emulatore accelerato al massimo) e non potrebbero mai produrre codice ottimizzato perché l’ottimizzatore sarebbe limitato dalla risorse della macchina 8-bit.</p>
<p>Faremo particolare riferimento ai seguenti <em>cross compilatori</em> <em>multi-target</em>:</p>

<table>
<thead>
<tr>
<th>Architettura</th>
<th>Compilatore/Dev-Kit</th>
<th>Pagina</th>
</tr>
</thead>
<tbody>
<tr>
<td>Intel 8080</td>
<td>ACK</td>
<td><a href="https://github.com/davidgiven/ack">https://github.com/davidgiven/ack</a></td>
</tr>
<tr>
<td>MOS 6502</td>
<td>CC65</td>
<td><a href="https://github.com/cc65/cc65">https://github.com/cc65/cc65</a></td>
</tr>
<tr>
<td>Motorola 6809</td>
<td>CMOC</td>
<td><a href="https://perso.b2b2c.ca/~sarrazip/dev/cmoc.html">https://perso.b2b2c.ca/~sarrazip/dev/cmoc.html</a></td>
</tr>
<tr>
<td>Zilog 80</td>
<td>SCCZ80/ZSDCC (Z88DK)</td>
<td><a href="https://github.com/z88dk/z88dk">https://github.com/z88dk/z88dk</a></td>
</tr>
</tbody>
</table><p>Inoltre esistono altri <em>cross compilatori</em> C <em>multi-target</em> che non tratteremo qui ma per i quali buona parte delle stesse tecniche generiche rimangono valide:</p>
<ul>
<li>LCC1802 (<a href="https://sites.google.com/site/lcc1802/">https://sites.google.com/site/lcc1802/</a>) per il COSMAC 1802;</li>
<li>SDCC (<a href="http://sdcc.sourceforge.net/">http://sdcc.sourceforge.net/</a>) per svariate architetture di microprocessori come lo Z80 e di microcontrollori come l’Intel 8051;</li>
<li>GCC-6809 (<a href="https://github.com/bcd/gcc">https://github.com/bcd/gcc</a>) per 6809 (adattamento di GCC);</li>
<li>GCC-6502 (<a href="https://github.com/itszor/gcc-6502-bits">https://github.com/itszor/gcc-6502-bits</a>) per 6502 (adattamento di GCC);</li>
<li>SmallC-85 (<a href="https://github.com/ncb85/SmallC-85">https://github.com/ncb85/SmallC-85</a>) per Intel 8080/8085 ;</li>
<li>devkitSMS (<a href="https://github.com/sverx/devkitSMS">https://github.com/sverx/devkitSMS</a>) per le console Sega basate su Z80 come Sega Master System, Sega Game Gear e Sega SG-1000.</li>
</ul>
<p>Si noti come il dev-kit Z88DK disponga di due compilatori:</p>
<ul>
<li>l’affidabile SCCZ80 che è anche molto veloce nelle compilazione,</li>
<li>lo sperimentale ZSDCC (versione ottimizzata per Z80 di SDCC sopracitato) che però può produrre codice più efficiente e compatto di SCCZ80 a costo di compilazione più lenta e rischio di introdurre bug.</li>
</ul>
<p>Quasi tutti i compilatori che stiamo prendendo in considerazione generano codice per una sola architettura (sono <em>mono-architettura</em>) pur essendo <em>multi-target</em>. ACK è una eccezione essendo anche <em>multi-architettura</em> (con supporto per Intel 8080, Intel 8088/8086, I386, 68K, MIPS, PDP11, etc.).</p>
<p>Questo articolo <strong>non</strong> è né una introduzione né un manuale d’uso di questi compilatori e <strong>non</strong> tratterà:</p>
<ul>
<li>l’istallazione dei compilatori;</li>
<li>elementi specifici per l’uso di base di un compilatore</li>
</ul>
<p>Per i dettagli sull’istallazione e l’uso di base dei compilatori in questione, facciamo riferimento ai manuali e alle pagine web dei relativi compilatori.</p>
<p><strong>Sottoinsieme di ANSI C</strong><br>
In questo articolo per ANSI C intendiamo sostanzialmente un grosso sotto-insieme dello standard C89 in cui i <code>float</code> e i <code>long long</code> sono opzionali ma i puntatori a funzioni e puntatori a <code>struct</code> sono presenti.<br>
Non stiamo considerando versioni precedenti del C come per esempio C in sintassi <em>K&amp;R</em>.</p>
<h2 id="motivazione">Motivazione</h2>
<p>Per quale motivo dovremmo usare il C per programmare dei sistemi 8-bit?<br>
Tradizionalmente queste macchine vengono programmate in Assembly o in BASIC interpretato o in un mix dei due.<br>
Data la limitatezza delle risorse è spesso necessario ricorrere all’Assembly. Il BASIC è invece comodo per la sua semplicità e perché spesso un interprete è già presente sulla macchina.</p>
<p>Volendo limitare il confronto a questi soli tre linguaggi il seguente schema ci dà una idea delle ragione per l’uso del C.</p>

<table>
<thead>
<tr>
<th></th>
<th>facilità</th>
<th>portabilità</th>
<th>efficienza</th>
</tr>
</thead>
<tbody>
<tr>
<td>BASIC</td>
<td>SI</td>
<td>parziale</td>
<td>poca</td>
</tr>
<tr>
<td>Assembly</td>
<td>NO</td>
<td>NO</td>
<td>ottima</td>
</tr>
<tr>
<td>C</td>
<td>SI</td>
<td>SI</td>
<td>buona</td>
</tr>
</tbody>
</table><h3 id="portabilità-estrema">Portabilità estrema</h3>
<p>Quindi la ragione principale per l’uso del C è la sua portabilità assoluta. In particolare se si usa un sottoinsieme dell’ANSI C che è uno standard.<br>
In particolare l’ANSI C ci pemette di:</p>
<ul>
<li>fare porting semplificato tra una architettura all’altra</li>
<li>scrivere codice “universale”, cioè valido per diversi target <strong>senza</strong> alcuna modifica</li>
</ul>
<h3 id="buone-performance">Buone performance</h3>
<p>Qualcuno si spinge a dichiarare che il C sia una sorta di Assembly universale. Questa è una affermazione un po’ troppo ottimistica perché del C scritto molto bene non batterà mai dell’Assembly scritto bene.<br>
Ciò nonostante il C è probabilmente il linguaggio più vicino all’Assembly tra i linguaggi che permettono anche la programmazione ad alto livello.</p>
<h3 id="controindicazioni-sentimentali">Controindicazioni “sentimentali”</h3>
<p>Una ragione non-razionale ma “sentimentale” per non usare il C sarebbe data dal fatto che il C è sicuramente meno <em>vintage</em> del BASIC e Assembly perché non era un linguaggio comune sugli home computer degli anni 80 (ma lo era sui computer professionali 8-bit come sulle macchine che usavano il sistema operativo CP/M).<br>
Credo che la programmazione in C abbia però il grosso vantaggio di poterci fare programmare l’hardware di quasi tutti i sistemi 8-bit.</p>
<h2 id="scrivere-codice-portabile">Scrivere codice portabile</h2>
<p>Scrivere codice facilmente portabile o addirittura direttammente compilabile per diverse piattaforme è possibile in C attraverso varie strategie:</p>
<ul>
<li>Scrivere codice <em>agnostico</em> dell’hardware e che quindi usi <em>interfacce astratte</em> (cioè delle API indipendenti dall’hardware).</li>
<li>Usare implementazioni diverse per le <em>interfacce</em> comuni da selezionare al momento della compilazione (per esempio attraverso <em>direttive al precompilatore</em> o fornendo file diversi al momento del linking).</li>
</ul>
<h3 id="sistemi-supportati-dai-compilatori">Sistemi supportati dai compilatori</h3>
<p>Questo diventa banale se il nostro dev-kit multi-target mette a disposizione una libreria multi-target o se ci si limita a usare le librerie standard del C (stdio, stdlib, etc.). Se si è in queste condizioni, allora basterà ricompilare il codice per ogni target e la libreria multi-target del del dev-kit farà la “magia” per noi.</p>
<p>Solo CC65 e Z88DK propongono interfacce multi-target per input e output oltre le librerie C standard:</p>

<table>
<thead>
<tr>
<th>Dev-Kit</th>
<th>Architettura</th>
<th>librerie multi-target</th>
</tr>
</thead>
<tbody>
<tr>
<td>Z88DK</td>
<td>Zilog Z80</td>
<td>standard C lib, conio, vt52, vt100, sprite software, UDG, bitmap</td>
</tr>
<tr>
<td>CC65</td>
<td>MOS 6502</td>
<td>standard C lib, conio, tgi (bitmap)</td>
</tr>
<tr>
<td>CMOC</td>
<td>Motorola 6809</td>
<td>standard C lib</td>
</tr>
<tr>
<td>ACK</td>
<td>Intel 8080</td>
<td>standard C lib</td>
</tr>
</tbody>
</table><p>In particolare Z88DK possiede strumenti potentissimi per la grafica multi-target (solo su Z80) e fornisce diverse API sia per gli sprite software (<a href="https://github.com/z88dk/z88dk/wiki/monographics">https://github.com/z88dk/z88dk/wiki/monographics</a>) che per i caratteri ridefiniti per buona parte dei suoi 80 target.</p>
<p><strong><em>Esempio</em></strong>:  Il gioco multi-piattaforma H-Tron è un esempio (<a href="https://sourceforge.net/projects/h-tron/">https://sourceforge.net/projects/h-tron/</a>) in cui si usano le API previste dal dev-kit Z88DK per creare un gioco su molti sistemi basati sull’architettura Z80.</p>
<p>Quindi se usassimo esclusivamente le librerie standard C potremmo avere codice compilabile con ACK, CMOC, CC65 e Z88DK. Mentre se usassimo anche <em>conio</em> avremmo codice compilabile per <em>CC65</em> e <em>Z88DK</em>.</p>
<p>In tutti gli altri casi se vogliamo scrivere codice portabile su architetture e sistemi diversi bisognerà costruirsi delle API. Sostanzialmente si deve creare un <em>hardware abstraction layer</em> che permette di <strong>separare</strong></p>
<ul>
<li>il codice che non dipende dall’hardware (per esempio la logica di un gioco)</li>
<li>dal codice che dipende dall’hardware (per esempio l’input, output in un gioco).</li>
</ul>
<p>Questo <em>pattern</em> è assai comune nella programmazione moderna e non è una esclusiva del C ma il C fornisce una serie di strumenti utili per implementare questo <em>pattern</em> in maniera che che si possano supportare hardware diversi da selezione al momento della compilazione. In particolare il C prevede un potente precompilatore con comandi come:</p>
<ul>
<li><code>#define</code> -&gt; per definire una macro</li>
<li><code>#if</code> … <code>defined(...)</code> … <code>#elif</code> … <code>#else</code> -&gt; per selezione porzioni di codice che dipendono dal valore o esistenza di una macro.</li>
</ul>
<p>Inoltre tutti i compilatori prevedono una opzione (in genere <code>-D</code>) per passare una variabile al precompilatore con eventuale valore. Alcuni compilatori come CC65 implicitamente definiscono una variabile col nome del target (per esempio <em><strong>VIC20</strong></em>) per il quale si intende compilare.</p>
<p>Nel codice avremo qualcosa come:</p>
<pre><code>...
		#elif defined(__PV1000__)
			#define XSize 28
		#elif defined(__OSIC1P__) || defined(__G800__) || defined(__RX78__) 
			#define XSize 24
		#elif defined(__VIC20__) 
			#define XSize 22
...
</code></pre>
<p>per cui al momento di compilare per il <em>Vic 20</em> il precompilatore selezionerà per noi la definizione di <code>XSize</code> specifica del <em>Vic 20</em>.</p>
<p>Questo permette al precompilatore non solo di selezionare le parti di codice specifiche per una macchina, ma anche di selezionare opzioni specifiche per configurazione delle macchina (memoria aggiuntiva, scheda grafica aggiuntivo, modo grafica, compilazione di debug, etc.).</p>
<p>Come esempio principale faremo riferimento al progetto <em>Cross-Chase</em>:<br>
<a href="https://github.com/Fabrizio-Caruso/CROSS-CHASE">https://github.com/Fabrizio-Caruso/CROSS-CHASE</a></p>
<p>Il codice di <em>Cross-Chase</em> fornisce un esempio su come scrivere codice <em>universale</em> valido per qualsiasi sistema ed architettura:</p>
<ul>
<li>il codice del gioco (directory <em>src/chase</em>) è indipendente dall’hardware</li>
<li>il codice della libreria <em>crossLib</em> (directory <em>src/cross_lib</em>) implementa i dettagli di ogni hardware possibile</li>
</ul>
<h3 id="sistemi-non-supportati">Sistemi non supportati</h3>
<p>I nostri dev-kit supportano una lista di target per ogni architettura attraverso la presenza di librerie specifiche per l’hardware. E’ comunque possibile sfruttare questi dev-kit per altri target con la stessa architettura ma dovremo fare più lavoro e saremo costretti ad implementare tutta la parte di codice specifica del target:</p>
<ul>
<li>codice necessario per gestire l’input/output (grafica, tastiera, joystick, suoni, etc.)</li>
<li>codice necessario per inizializzare correttamente il binario</li>
</ul>
<p>Per fare ciò potremo in molti casi usare le routine già presenti nella ROM (per un esempio si veda la sezione di questo articolo che tratta l’uso delle routine della ROM).</p>
<p>Inoltre dovremmo anche usare dei convertitori del binario in un formato accettabile per il nuovo sistema (e potremmo essere costretti a doverli scrivere qualora non siano già a disposizione).</p>
<p>Potremo quindi scrivere codice portabile anche a questi sistemi.</p>
<p>Per esempio CC65 non supporta <em>BBC Micro</em> e <em>Atari 7800</em> e CMOC non supporta <em>Olivetti Prodest PC128</em> ma è comunque possibile usare i dev-kit per produrre binari per questi target:</p>
<ul>
<li>Cross Chase (<a href="https://github.com/Fabrizio-Caruso/CROSS-CHASE">https://github.com/Fabrizio-Caruso/CROSS-CHASE</a>) supporta (in principio) qualunque architettura anche non supportata direttamente dai compilatori come per esempio l’Olivetti Prodest PC128.</li>
<li>Il gioco Robotsfindskitten è stato portato per l’Atari 7800 usando CC65 (<a href="https://sourceforge.net/projects/rfk7800/files/rfk7800/">https://sourceforge.net/projects/rfk7800/files/rfk7800/</a>).</li>
<li>BBC è stato aggiunto come target sperimentale su CC65 (<a href="https://github.com/dominicbeesley/cc65">https://github.com/dominicbeesley/cc65</a>).</li>
</ul>
<h4 id="compilare-per-sistemi-non-supportati">Compilare per sistemi non supportati</h4>
<p>Qui diamo una lista delle opzioni di compilazione per target generico per ogni dev-kit in maniera da compilare per una data architettura senza alcuna dipendenza da un target specifico. Per maggiori dettagli facciamo riferimento ai rispettivi manuali.</p>

<table>
<thead>
<tr>
<th>Architettura</th>
<th>Dev-Kit</th>
<th>Opzione</th>
</tr>
</thead>
<tbody>
<tr>
<td>Intel 8080</td>
<td>ACK</td>
<td>(*)</td>
</tr>
<tr>
<td>MOS 6502</td>
<td>CC65</td>
<td><code>+none</code></td>
</tr>
<tr>
<td>Motorola 6809</td>
<td>CMOC</td>
<td><code>--nodefaultlibs</code></td>
</tr>
<tr>
<td>Zilog 80</td>
<td>SCCZ80/ZSDCC (Z88DK)</td>
<td><code>+test</code>, <code>+embedded</code> (nuova libreria),  <code>+cpm</code> (per vari sistemi CP/M)</td>
</tr>
</tbody>
</table><p>(*) ACK prevede solo il target CP/M-80 per l’architettura Intel 8080 ma è possibile almeno in principio usare ACK per produrre binari Intel 8080 generico ma non è semplice in quanto ACK usa una sequenze da di comandi per produrre il Intel 8080 partendo dal C e passando da vari stai intermedi compreso un byte-code “EM”.<br>
Qui di seguito listo i comandi utili:</p>
<ol>
<li><code>ccp.ansi</code>:  precompilatore del C</li>
<li><code>em_cemcom.ansi</code>: compila C preprocessato producendo bytecode</li>
<li><code>em_opt</code>: ottimizza il bytecode</li>
<li><code>cpm/ncg</code>: genera Assembly da bytecode</li>
<li><code>cpm/as</code>: genera codice Intel 80 da Assembly</li>
<li><code>em_led</code>: linker</li>
</ol>
<h2 id="ottimizzare-il-codice-in-generale">Ottimizzare il codice in generale</h2>
<p>Ci sono alcune regole generali per scrivere codice migliore indipendentemente dal fatto che l’architettura sia 8-bit o meno.</p>
<h3 id="riutilizziamo-le-stesse-funzioni">Riutilizziamo le stesse funzioni</h3>
<p>In generale, in qualunque linguaggio di programmazione si voglia programmare, è importante evitare la duplicazione del codice o la scrittura di codice superfluo.</p>
<h4 id="programmazione-strutturata">Programmazione strutturata</h4>
<p>Spesso guardando bene le funzioni che abbiamo scritto scopriremo che condividono delle parti comuni e che quindi potremo <em>fattorizzare</em> costruendo delle <em>sotto-funzioni</em> che le nostre funzioni chiameranno.<br>
Dobbiamo però tenere conto che, oltre un certo limite, una eccessiva granularità del codice ha effetti deleteri perché una chiamata ad una funzione ha un costo computazionale e di memoria.</p>
<h4 id="generalizzare-il-codice-parametrizzandolo">Generalizzare il codice parametrizzandolo</h4>
<p>In alcuni casi è possibile generalizzare il codice passando un parametro per evitare di scrivere due funzioni diverse molto simili.<br>
Un esempio si trova in <a href="https://github.com/Fabrizio-Caruso/CROSS-CHASE/blob/master/src/chase/character.h">https://github.com/Fabrizio-Caruso/CROSS-CHASE/blob/master/src/chase/character.h</a> dove, dato uno <code>struct</code> con due campi <code>_x</code> e <code>_y</code>,  vogliamo potere agire sul valore di uno o dell’altro in situazioni diverse:</p>
<pre><code>	struct CharacterStruct
	{
		unsigned char _x;
		unsigned char _y;
		...
	};
	typedef struct CharacterStruct Character;
</code></pre>
<p>Possiamo evitare di scrivere due diverse funzioni per agire su <code>_x</code> e su <code>_y</code> creando una unica funzione a cui si passa un <em>offset</em> che faccia da selettore:</p>
<pre><code>	unsigned char moveCharacter(Character* hunterPtr, unsigned char offset)
	{
		if((unsigned char) * ((unsigned char*)hunterPtr+offset) &lt; ... )
		{
			++(*((unsigned char *) hunterPtr+offset));
		}
		else if((unsigned char) *((unsigned char *) hunterPtr+offset) &gt; ... )
		{
			--(*((unsigned char *) hunterPtr+offset));
		}
	...
	}
</code></pre>
<p>Nel caso sopra stiamo sfruttando il fatto che il secondo campo <code>_y</code> si trova esattamente un byte dopo il primo campo <code>_x</code>. Quindi con <code>offset=0</code> accediamo al campo <code>_x</code> e con <code>offset=1</code> accediamo al campo <code>_y</code>.</p>
<p><strong>Avvertenze</strong>: Dobbiamo però considerare sempre che aggiungere un parametro ha un costo e quindi dovremo verificare (anche guardando la taglia del binario ottenuto) se nel nostro caso ha un costo inferiore al costo di una funzione aggiuntiva.</p>
<h4 id="stesso-codice-su-oggetti-simili">Stesso codice su <em>oggetti</em> simili</h4>
<p>Si può anche fare di più e usare lo stesso codice su <em>oggetti</em> che non sono esattamente dello stesso tipo ma che condividono solo alcuni aspetti comuni per esempio sfruttando gli <code>offset</code> dei campi negli <code>struct</code>, <em>puntatori a funzioni</em>, etc.<br>
Questo è possibile in generale tramite la <em>programmazione ad oggetti</em> di cui descriviamo una implementazione leggera per gli 8-bit in una sezione successiva.</p>
<h3 id="pre-incrementodecremento-vs-post-incrementodecremento">Pre-incremento/decremento vs Post-incremento/decremento</h3>
<p>Bisogna evitare operatori di post-incremento/decremento (<code>i++</code>, <code>i--</code>) quando non servono (cioè quando non serve il valore pre-incremento) e sostituirli con (<code>++i</code>, <code>--i</code>).<br>
Il motivo è che l’operatore di post-incremento richiede almeno una operazione in più dovendo conservare il valore originario.<br>
Nota: E’ totalmente inutile usare un operatore di post-incremento in un ciclo <code>for</code>.</p>
<h3 id="costanti-vs-variabili">Costanti vs Variabili</h3>
<p>Una qualunque architettura potrà ottimizzare meglio del codici in cui delle variabili sono sostituite con delle costanti.</p>
<h4 id="usiamo-costanti">Usiamo costanti</h4>
<p>Quindi se una data variabile ha un valore noto al momento della compilazione, è importante che sia rimpiazzata con una costante.<br>
Se il suo valore, pur essendo noto al momento della compilazione, dovesse dipende da una opzione di compilazione, allora la sostituiremo con una <em>macro</em> da settare attraverso una opzione di compilazione, in maniera tale che sia trattata come una costante dal compilatore.</p>
<h4 id="aiutiamo-il-compilatore-a-ottimizzare-le-costanti">Aiutiamo il compilatore a ottimizzare le costanti</h4>
<p>Inoltre, per compilatori <em>single pass</em> (come la maggioranza dei cross-compilatori 8-bit come per esempio CC65), può essere importante aiutare il compilatore a capire che una data espressione sia una costante.</p>
<p><strong><em>Esempio</em></strong> (preso da <a href="https://www.cc65.org/doc/coding.html">https://www.cc65.org/doc/coding.html</a>):<br>
Un compilatore <em>single pass</em> valuterà la seguente espressione da sinistra a destra non capendo che <code>OFFS+3</code> è una costante.</p>
<pre><code>	#define OFFS   4
	int  i;
	i = i + OFFS + 3;
</code></pre>
<p>Quindi sarebbe meglio riscrivere <code>i = i + OFFS+3</code> come <code>i = OFFS+3+i</code> oppure <code>i = i + (OFFS+3)</code>.</p>
<h2 id="ottimizzare-il-codice-per-gli-8-bit">Ottimizzare il codice per gli 8-bit</h2>
<p>Il C è un linguaggio che presenta sia costrutti ad alto livello (come <code>struct</code>, le funzioni come parametri, etc.) sia costruiti a basso livello (come i puntatori e la loro manipolazione). Questo non basta per farne un linguaggio direttamente adatto alla programmazione su macchine 8-bit.</p>
<h3 id="implementare-peek-e-poke-in-c">Implementare <code>peek</code> e <code>poke</code> in C</h3>
<p>Quasi sicuramente avremo bisogno di fare scrivere e leggere dei singoli byte su alcune specifiche locazioni di memoria.<br>
Il modo per fare questo è in BASIC sarebbe attraverso i comando <code>peek</code> e <code>poke</code>.<br>
In C dobbiamo farlo attraverso dei puntatori la cui sintassi non è legibilissima. Potremo però costruirci delle utili macro che useremo nel nostro codice:</p>
<pre><code>    #define POKE(addr,val)  (*(unsigned char*) (addr) = (val))
    #define PEEK(addr)      (*(unsigned char*) (addr))
</code></pre>
<p>Nota: I compilatori scriveranno codice ottimale nel caso in cui si passino delle costanti come parametri.</p>
<p>Per maggiori dettagli facciamo riferimento a: <a href="https://github.com/cc65/wiki/wiki/PEEK-and-POKE">https://github.com/cc65/wiki/wiki/PEEK-and-POKE</a></p>
<h3 id="i-tipi-migliori-per-gli-8-bit">I “tipi migliori” per gli 8-bit</h3>
<p>Una premessa importante per la scelta dei tipi da preferire per architettura è data dal fatto che in generale abbiamo questa situazione:</p>
<ul>
<li>tutte le operazioni aritmetiche sono solo a 8 bit</li>
<li>la maggior parte delle operazioni sono ad 8 bit, alcune sono a 16-bit e nessuna operazione è a 32 bit</li>
<li>le operazioni <code>signed</code> (cioè con segno) sono più lente di quelle <code>unsigned</code></li>
<li>l’hardware non supporta operazioni in <em>virgola mobile</em></li>
</ul>
<h4 id="tipi-interi-vs-tipi-a-virgola-mobile">Tipi interi vs tipi a virgola mobile</h4>
<p>Il C prevede tipi numerici interi con segno (<code>char</code>, <code>short</code>, <code>int</code>, <code>long</code>, <code>long long</code> e loro equivalenti in versione <code>unsigned</code>).<br>
Molti compilatori (ma non CC65) prevedono il tipo <code>float</code> (numeri a <em>virgola mobile</em>) che qui non tratteremo. Bisogna considerare che i <code>float</code> delle architetture 8-bit sono tutti <em>software float</em> ed hanno quindi un costo computazionale notevole. Sarebbero quindi da usare solo se strettamente necessari.</p>
<h4 id="il-nostro-amico-unsigned">Il nostro amico <em>unsigned</em></h4>
<p>Siccome le architetture 8-bit che stiamo considerandno <strong>NON</strong> gestiscono ottimalmente tipi <code>signed</code>, dobbiamo evitare il più possibile l’uso di tipi numerici <code>signed</code>.</p>
<h4 id="size-matters">“Size matters!”</h4>
<p>La dimensione dei tipi numeri standard dipende dal compilatore e dall’architettura e non dal linguaggio.<br>
Più recentemente sono stati introdotti dei tipi che fissano la dimensione in modo univoco (come per esempio <code>uint8_t</code> per l’intero <code>unsigend</code> a 8 bit).<br>
Il modo standard per includere questi tipi a taglia fissa</p>
<pre><code>	#include &lt;stdint.h&gt;
</code></pre>
<p>Non tutti i compilatori 8-bit dispongono di questi tipi.</p>
<p>Fortunatamente per la stragrande maggioranza dei compilatori 8-bit abbiamo la seguente situazione:</p>

<table>
<thead>
<tr>
<th>tipo</th>
<th>numero bit</th>
<th>equivalente</th>
</tr>
</thead>
<tbody>
<tr>
<td><code>unsigned char</code></td>
<td>8</td>
<td><code>uint8_t</code></td>
</tr>
<tr>
<td><code>unsigned short</code></td>
<td>16</td>
<td><code>uint16_t</code></td>
</tr>
<tr>
<td><code>unsigned int</code></td>
<td>16</td>
<td><code>uint16_t</code></td>
</tr>
<tr>
<td><code>unsigned long</code></td>
<td>32</td>
<td><code>uint32_t</code></td>
</tr>
</tbody>
</table><p>Quindi dovremo:</p>
<ul>
<li>usare il più possibile <code>unsigned char</code> (o <code>uint8_t</code>) per le operazioni aritmetiche;</li>
<li>usare <code>unsigned char</code> (o <code>uint8_t</code>) e <code>unsigned short</code> (o <code>uint16_t</code>) per tutte le altre operazioni, evitando se possibile qualunque operazione a 32 bit.</li>
</ul>
<p>Nota: In assenza di tipi con dimensione fissata, sarebbe una buona pratica creare dei <code>typedef</code> opportuni:</p>
<pre><code>	typedef unsigned char uint8_t;
	typedef unsigned short uint16_t;
	typedef unsigned long uint32_t;
</code></pre>
<h3 id="scelta-delle-operazioni">Scelta delle operazioni</h3>
<p>Quando scriviamo codice per una architettura 8-bit dobbiamo evitare se possibile codice con operazioni inefficienti o che ci obblighino a usare tipi non adatti (come i tipi <code>signed</code> o tipi a 16 o peggio 32 bit).</p>
<h4 id="non-produciamo-signed">Non produciamo <em>signed</em></h4>
<p>In particolare, se possibile, spesso si può riscrivere il codice in maniera da evitare sottrazioni e quando questo non è possibile, si può almeno fare in modo che il risultato della sottrazione sia sempre non-negativo.</p>
<h4 id="evitiamo-i-prodotti-espliciti">Evitiamo i prodotti espliciti</h4>
<p>Tutte le architetture che abbiamo preso in considerazione, con la sola esclusione di Motorola 6809, non dispongono di una operazione per effettuare il prodotto di due valori a 8 bit.<br>
Quindi, se possibile dobbiamo evitare i prodotti adattando il nostro codice, oppure limitarci a prodotti e divisioni che siano potenze di 2 e implementandoli con operazioni di shift con gli operatori <em>&lt;&lt;</em> e <em>&gt;&gt;</em>:</p>
<pre><code>	unsigned char foo, bar;
	...
	foo &lt;&lt; 2; // moltiplicazione per 2^2=4
	bar &gt;&gt; 1; // divisione per 2^1=2
</code></pre>
<h4 id="riscrivere-certe-operazioni">Riscrivere certe operazioni</h4>
<p>Molte operazioni come il modulo possono essere riscritte in maniera più efficiente per gli 8 bit usando operatori bit a bit. Non sempre il compilatore ottimizza nel modo migliore. Quando il compilatore non ce la fa, dobbiamo dargli una mano noi:</p>
<pre><code>	unsigned char foo;
	...
	if(foo&amp;1) // equivalente a foo%2
	{
		...
	}
</code></pre>
<h3 id="variabili-e-parametri">Variabili e parametri</h3>
<p>Uno dei più grossi limiti dell’architettura MOS 6502 non è la penuria di registri come si potrebbe pensare ma è la dimensione limitata del suo <em>stack hardware</em> (in <em>pagina uno</em>: <code>$0100-01FF</code>) che lo rende inutilizzabile in C per la gestioni dello <em>scope</em> delle variabili e i parametri delle funzioni.<br>
Quindi un compilatore ANSI C per 6502 sarà quasi sicuramente costretto a usare uno <em>stack software</em> per</p>
<ul>
<li>gestire lo scope delle variabili,</li>
<li>gestire il passaggio dei parametri.</li>
</ul>
<p>Le altre architetture 8-bit che stiamo considerando soffrono meno di questo problema ma la gestione delle scope delle variabili e dei parametri ha un costo anche quando si usa uno <em>stack hardware</em>.</p>
<h4 id="un-antipattern-può-aiutarci">Un <em>antipattern</em> può aiutarci</h4>
<p>Un modo per ridurre il problema è limitare l’uso delle variabili locali e dei parametri passati alle funzioni. Questo è chiaramente un <em>antipattern</em> e se lo applicassimo a tutto il nostro codice otterremo il classico <em>spaghetti code</em>. Dobbiamo quindi scegliere sapientemente quali variabili sono assolutamente locali e quali possono essere usate come globali. Avremo codice meno generico di quello che avremmo voluto ma sarà più efficiente. <strong>NON</strong> sto suggerendo di rendere tutte le variabili globali e di non passare mai parametri alle funzioni.</p>
<h4 id="usare-funzioni-non-re-entrant">[6502] Usare funzioni non re-entrant</h4>
<p>Il compilatore CC65 per l’architettura MOS 6502 mette a disposizione l’opzione <code>-Cl</code> che rende tutte le variabili locali come <code>static</code>, quindi globali. Questo ha l’effetto di evitare l’uso dello <em>stack software</em> per il loro scope. Ha però l’effetto di rendere tutte le nostre funzioni non re-entrant. In pratica questo ci impedisce di usare funzioni recursive. Questa non è un grave perdita perché la ricorsione sarebbe comunque una operazione troppo costosa per una architettura 8-bit.</p>
<h4 id="usare-la-pagina-zero">[6502] Usare la pagina zero</h4>
<p>Il C standard prevede la keyword <code>register</code> per suggerire al compilatore di mettere una variabile in un registro.<br>
In genere i compilatori moderni ignorano questa keyword perché lasciano questa scelta ai loro ottimizzatori. Questo è vero per i compilatori in questione ad eccezione di quello presenti in CC65 che la usa come suggerimento al compilatore per mettere una variabile in <em>pagina zero</em>. Il MOS 6502 accede in maniera più efficiente a tale pagina di memoria. Si può guadagnare memoria e velocità.<br>
Per quanto riguarda l’architettura MOS 6502, il sistema operativo di queste macchine usa una parte della pagina zero. Resta comunque una manciata di byte a disposizione del programmatore.<br>
CC65 per default lascia 6 byte della pagina zero a disposizione delle variabili dichiarate con keyword <code>register</code>.<br>
Potrebbe sembrare quindi ovvio dichiarare molte variabili come <code>register</code> ma <strong>NON</strong> è così semplice perché tutto ha un costo. Per mettere una variabile sulla <em>pagina zero</em> sono necessarie diverse operazioni. Quindi se ne avrà un vantaggio quando le variabili sono molto usate.<br>
In pratica i due scenari in cui è conveniente sono:</p>
<ol>
<li>parametri di tipo puntatore a <code>struct</code> usati almeno 3 volte all’interno di una funzione</li>
<li>variabile in un loop che si ripete almeno un centinaio di volte</li>
</ol>
<p>Un riferimento più preciso è dato da: <a href="https://www.cc65.org/doc/cc65-8.html">https://www.cc65.org/doc/cc65-8.html</a></p>
<p>Il mio consiglio è quello di compilare e vedere se il binario è divenuto più breve.</p>
<h3 id="struttura-ottimale-del-binario">Struttura ottimale del binario</h3>
<p>Se il nostro programma prevede dei dati in una definita area di memoria, sarebbe meglio metterli direttamente nel binario che verrà copiato in memoria durante il caricamento. Se questi dati sono invece nel codice, saremo costretti a scrivere del codice che li copia nell’area di memoria in cui sono previsti.<br>
Il caso più comune è forse quello degli sprites e dei caratteri/tiles ridefiniti.</p>
<p>Spesso (ma non sempre) le architetture basate su MOS 6502 prevedono video <em>memory mapped</em> in cui i dati della grafica si trovano nella stessa RAM a cui accede la CPU.</p>
<p>Molte architetture basate su Z80 (MSX, Spectravideo, Memotech, Tatung Einstein, etc.) usano il chip Texas VDP che invece ha una memoria video dedicata. Quindi non potremo mettere la grafica direttamente in questa memoria.</p>
<h4 id="cc65-istruiamo-il-linker">[CC65] Istruiamo il linker</h4>
<p>Ogni compilatore mette a disposizioni strumenti diversi per definire la struttura del binario e quindi permetterci di costruirlo in maniera che i dati siano caricati in una determinata zona di memoria durante il load del programma senza uso di codice aggiuntivo.<br>
In particolare su CC65 si può usare il file .cfg di configurazione del linker che descrive la struttura del binario che vogliamo produrre.<br>
Il linker di CC65 non è semplicissimo da configurare ed una descrizione andrebbe oltre lo scopo di questo articolo.<br>
Una descrizione dettagliata è presente su:<br>
<a href="https://cc65.github.io/doc/ld65.html">https://cc65.github.io/doc/ld65.html</a><br>
Il mio consiglio è di leggere il manuale e di modificare i file di default .cfg già presenti in CC65 al fine di adattarli al proprio use-case.</p>
<h5 id="exomizer-ci-aiuta-anche-in-questo-caso">Exomizer ci aiuta (anche) in questo caso</h5>
<p>In alcuni casi se la nostra grafica deve trovarsi in un’area molto lontana dal codice, e vogliamo creare un unico binario, avremo un binario enorme e con un “buco”. Questo è il caso per esempio del C64 in cui la grafica per caratteri e sprites può trovarsi lontana dal codice. In questo caso io suggerisco di usare <em>exomizer</em> sul risultato finale: <a href="https://bitbucket.org/magli143/exomizer/wiki/Home">https://bitbucket.org/magli143/exomizer/wiki/Home</a></p>
<h4 id="z88dk-appmake-fa-quasi-tutto-per-noi">[Z88DK] <em>Appmake</em> fa (quasi) tutto per noi</h4>
<p>Z88DK fa molto di più e il suo potente tool <em>appmake</em> costuisce dei binari nel formato richiesto.<br>
Z88DK consente comunque all’utente di definire sezioni di memoria e di definire il “packaging” del binario ma non è semplice.<br>
Questo argomento è trattato in dettaglio in<br>
<a href="https://github.com/z88dk/z88dk/issues/860">https://github.com/z88dk/z88dk/issues/860</a></p>
<h3 id="codice-su-file-diversi">Codice su file diversi?</h3>
<p>In generale è bene separare in più file il proprio codice se il progetto è di grosse dimensioni.<br>
Questa buona pratica può però avere degli effetti deleteri per gli ottimizzatori dei compilatori 8-bit perché in generale non eseguono <em>link-time optimization</em>, cioè non ottimizzeranno codice tra più file ma si limitano ad ottimizzare ogni file singolarmente.<br>
Quindi se per esempio abbiamo una funzione che chiamiamo una sola volta e la funzione è definita nello stesso file in cui viene usata, l’ottimizzatore potre metterla <em>in line</em> ma non lo farà se la funzione è definita in un altro file.<br>
Il mio consiglio <strong>non</strong> quello di creare file enormi con tutto ma è quello di tenere comunque conto di questo aspetto quando si decide di separare il codice su più file e di non abusare di questa buona pratica.</p>
<h2 id="programmazione-ad-oggetti">Programmazione ad oggetti</h2>
<p>Contrariamente a quello che si possa credere, la programmazione ad oggetti è possibile in ANSI C e può aiutarci a produrre codice più compatto in alcune situazioni. Esistono interi framework ad oggetti che usano ANSI C (es. Gnome è scritto usando <em>GObject</em> che è uno di questi framework).</p>
<p>Nel caso delle macchine 8-bit con vincoli di memoria molto forti, possiamo comunque implementare <em>classi</em>, <em>polimorfismo</em> ed <em>ereditarietà</em> in maniera molto efficiente.<br>
Una trattazione dettagliata non è possibile in questo articolo e qui ci limitiamo a citare i due strumenti fondamentali:</p>
<ul>
<li>usare <em>puntatori a funzioni</em> per ottenere metodi <em>polimorfici</em>, cioè il cui <em>binding</em> (e quindi comportamento) è dinamicamente definito a <em>run-time</em>. Si può evitare l’implementazione di una <em>vtable</em> se ci si limita a classi con un solo metodo polimorfico.</li>
<li>usare <em>puntatori a</em> <code>struct</code> e <em>composizione</em> per implementare sotto-classi: dato uno <code>struct</code> A, si implementa una sua sotto-classe con uno <code>struct</code> B definito come uno <code>struct</code> il cui <strong>primo</strong> campo è A. Usando puntatori a tali <code>struct</code>, il C garantisce che gli <em>offset</em> di B siano gli stessi degli offset di A.</li>
</ul>
<p>Esempio (preso da<br>
<a href="https://github.com/Fabrizio-Caruso/CROSS-CHASE/tree/master/src/chase">https://github.com/Fabrizio-Caruso/CROSS-CHASE/tree/master/src/chase</a>)<br>
Definiamo <code>Item</code> come un sotto-classe di <code>Character</code> a cui aggiungiamo delle variabili ed il metodo polimorfico <code>_effect()</code>:</p>
<pre><code>	struct CharacterStruct
	{
		unsigned char _x;
		unsigned char _y;
		unsigned char _status;
		Image* _imagePtr;
	};
	typedef struct CharacterStruct Character;
...
 	struct ItemStruct
	{
		Character _character;
		void (*_effect)(void);
		unsigned short _coolDown;
		unsigned char _blink;
	};
	typedef struct ItemStruct Item;
</code></pre>
<p>e poi potremo passare un puntatore a <code>Item</code> come se fosse un puntatore a <code>Character</code> (facendo un semplice <em>cast</em>):</p>
<pre><code>	Item *myItem;
	void foo(Character * aCharacter);
	...
	foo((Character *)myItem);
</code></pre>
<p>Perché ci guadagniamo in termine di memoria?<br>
Perché sarà possibile trattare più oggetti con lo stesso codice e quindi risparmiamo memoria.</p>
<h2 id="uso-avanzato-della-memoria">Uso avanzato della memoria</h2>
<p>Il compilatore C in genere produrrà un unico binario che conterrà codice e dati che verranno caricati in una specifica zona di memoria (è comunque possibile avere porzioni di codice non contigue).</p>
<p>In molte architetture alcune aree della memoria RAM sono usate come <em>buffer</em> oppure sono dedicate a usi specifici come alcune modalità grafiche.<br>
Il mio consiglio è quindi di studiare le mappa della memoria di ogni hardware per trovare queste preziose aree.<br>
Per esempio per il Vic 20: <a href="http://www.zimmers.net/cbmpics/cbm/vic/memorymap.txt">http://www.zimmers.net/cbmpics/cbm/vic/memorymap.txt</a></p>
<p>In particolare consiglio di cercare:</p>
<ul>
<li>buffer della cassetta, della tastiera, della stampante, del disco, etc.</li>
<li>memoria usata dal BASIC</li>
<li>aree di memoria dedicate a modi grafici (che non si intendono usare)</li>
<li>aree di memoria libere ma non contigue e che quindi non sarebbero parte del nostro binario</li>
</ul>
<p>Queste aree di memoria potrebbero essere sfruttate dal nostro codice se nel nostro use-case non servono per il loro scopo originario (esempio: se non intendiamo caricare da cassetta dopo l’avvio del programma, possiamo usare il buffer della cassetta per metterci delle variabili da usare dopo l’avvio potendolo comunque usare prima dell’avvio per caricare il nostro stesso programma da cassetta).</p>
<p><em>Esempi utili</em><br>
In questa tabella diamo alcuni esempi utili per sistemi che hanno poca memoria disponibile:</p>

<table>
<thead>
<tr>
<th>computer</th>
<th>descrizione</th>
<th>area</th>
</tr>
</thead>
<tbody>
<tr>
<td>Commodore 16</td>
<td>tape buffer</td>
<td>$0333-03F2</td>
</tr>
<tr>
<td>Commodore 16</td>
<td>BASIC input buffer</td>
<td>$0200-0258</td>
</tr>
<tr>
<td>Commodore 64 &amp; Vic 20</td>
<td>tape buffer</td>
<td>$033C-03FB</td>
</tr>
<tr>
<td>Commodore 64 &amp; Vic 20</td>
<td>BASIC input buffer</td>
<td>$0200-0258</td>
</tr>
<tr>
<td>Commodore Pet</td>
<td>system input buffer</td>
<td>$0200-0250</td>
</tr>
<tr>
<td>Commodore Pet</td>
<td>tape buffer</td>
<td>$033A-03F9</td>
</tr>
<tr>
<td>Galaksija</td>
<td>variable a-z</td>
<td>$2A00-2A68</td>
</tr>
<tr>
<td>Sinclair Spectrum 16K/48K</td>
<td>printer buffer</td>
<td>$5B00-5BFF</td>
</tr>
<tr>
<td>Mattel Aquarius</td>
<td>random number space</td>
<td>$381F-3844</td>
</tr>
<tr>
<td>Mattel Aquarius</td>
<td>input buffer</td>
<td>$3860-38A8</td>
</tr>
<tr>
<td>Oric</td>
<td>alternate charset</td>
<td>$B800-B7FF</td>
</tr>
<tr>
<td>Oric</td>
<td>grabable memory per modo hires</td>
<td>$9800-B3FF</td>
</tr>
<tr>
<td>Oric</td>
<td>Page 4</td>
<td>$0400-04FF</td>
</tr>
<tr>
<td>Sord M5</td>
<td>RAM for ROM routines (*)</td>
<td>$7000-$73FF</td>
</tr>
<tr>
<td>TRS-80 Model I/III/IV</td>
<td>RAM for ROM routines (*)</td>
<td>$4000-41FF</td>
</tr>
<tr>
<td>VZ200</td>
<td>printer buffer &amp; misc</td>
<td>$7930-79AB</td>
</tr>
<tr>
<td>VZ200</td>
<td>BASIC line input buffer</td>
<td>$79E8-7A28</td>
</tr>
</tbody>
</table><p>(*): Multiple buffer and auxiliary ram for ROM routiens. For more details please refer to:<br>
<a href="http://m5.arigato.cz/m5sysvar.html">http://m5.arigato.cz/m5sysvar.html</a> and <a href="http://www.trs-80.com/trs80-zaps-internals.htm">http://www.trs-80.com/trs80-zaps-internals.htm</a></p>
<p>In C standard potremmo solo definire le variabili puntatore e gli array come locazioni in queste aree di memoria.</p>
<p>Di seguito diamo un esempio di mappatura delle variabili a partire da <code>0xC000</code> in cui abbiamo definito uno <code>struct</code> di tipo <code>Character</code> che occupa 5 byte, e abbiamo le seguenti variabili:</p>
<ul>
<li><code>player</code> di tipo <code>Character</code>,</li>
<li><code>ghosts</code> di tipo <code>array</code> di 8 <code>Character</code> (40=$28 byte)</li>
<li><code>bombs</code> di tipo array di 4 <code>Character</code> (20=$14 byte)</li>
</ul>
<pre><code>	Character *ghosts = 0xC000;
	Character *bombs = 0xC000+$28;
	Character *player = 0xC000+$28+$14;
</code></pre>
<p>Questa soluzione generica con puntatori non sempre produce il codice ottimale perché obbliga a fare diverse <em>deferenziazioni</em> e comunque crea delle variabili puntatore (ognuna delle quali dovrebbe occupare 2 byte) che il compilatore potrebbe comunque allocare in memoria.</p>
<p>Non esiste un modo standard per dire al compilatore di mettere qualunque tipo di variabile in una specifica area di memoria.<br>
I compilatori di CC65 e Z88DK invece prevedono una sintassi per permetterci di fare questo e guadagnare diverse centinaia o migliaia di byte preziosi.<br>
Vari esempi sono presenti in:<br>
<a href="https://github.com/Fabrizio-Caruso/CROSS-CHASE/tree/master/src/cross_lib/memory">https://github.com/Fabrizio-Caruso/CROSS-CHASE/tree/master/src/cross_lib/memory</a></p>
<p>In particolare bisogna creare un file Assembly .s (con CC65) o .asm (con Z88DK) da linkare al nostro eseguibile in cui assegnamo un indirizzo ad ogni nome di variabile a cui  <strong>aggiungiamo</strong> un prefisso <em>underscore</em>.</p>
<p>Sintassi CC65 (esempio Vic 20)</p>
<pre><code>	.export _ghosts;
	_ghosts = $33c
	.export _bombs;
	_bombs = _ghosts + $28 
	.export _player;
	_player = _bombs + $14
</code></pre>
<p>Sintassi Z88DK (esempio Galaksija)</p>
<pre><code>	PUBLIC _ghosts, _bombs, _player
	defc _ghosts = 0x2A00
	defc _bombs = _ghosts + $28 
	defc _player = _bombs + $14
</code></pre>
<p>CMOC mette a disposizione l’opzione <code>--data=&lt;indirizzo&gt;</code> che permette di allocare tutte le variabili globali scrivibili a partire da un indirizzo dato.</p>
<p>La documentazione di ACK non dice nulla a riguardo. Potremo comunque definire i tipi puntatore e gli array nelle zone di memoria libera.</p>
<h2 id="compilazione-ottimizzata">Compilazione ottimizzata</h2>
<p>Non tratteremo in modo esaustivo le opzioni di compilazione dei cross-compilatori e consigliamo di fare riferimento ai loro rispettivi manuali per dettagli avanzati. Qui daremo una lista delle opzioni per compilare codice ottimizzato su ognuno dei compilatori che stiamo trattando.</p>
<h3 id="ottimizzazione-aggressiva">Ottimizzazione “aggressiva”</h3>
<p>Le seguenti opzioni applicano il massimo delle ottimizzazioni per produrre codice veloce e soprattutto compatto:</p>

<table>
<thead>
<tr>
<th>Architettura</th>
<th>Compilatore</th>
<th>Opzioni</th>
</tr>
</thead>
<tbody>
<tr>
<td>Intel 8080</td>
<td>ACK</td>
<td><code>-O6</code></td>
</tr>
<tr>
<td>Zilog Z80</td>
<td>SCCZ80 (Z88DK)</td>
<td><code>-O3</code></td>
</tr>
<tr>
<td>Zilog Z80</td>
<td>ZSDCC (Z88DK)</td>
<td><code>-SO3</code> <code>--max-alloc-node20000</code></td>
</tr>
<tr>
<td>MOS 6502</td>
<td>CC65</td>
<td><code>-O</code> <code>-Cl</code></td>
</tr>
<tr>
<td>Motorola 6809</td>
<td>CMOC</td>
<td><code>-O2</code></td>
</tr>
</tbody>
</table><h4 id="velocità-vs-memoria">Velocità vs Memoria</h4>
<p>In generale su molti target 8-bit il problema maggiore è la presenza di poca memoria per codice e dati. In generale il codice ottimizzato per la velocità sarà sia compatto che veloce ma non sempre le due cose andranno assieme.<br>
In alcuni altri casi l’obiettivo principale può essere la velocità anche a discapito della memoria.<br>
Alcuni compilatori mettono a disposizioni delle opzioni per specificare la propria preferenza tra velocità e memoria:</p>

<table>
<thead>
<tr>
<th>Architettura</th>
<th>Compilatore</th>
<th>Opzioni</th>
<th>Descrizione</th>
</tr>
</thead>
<tbody>
<tr>
<td>Zilog Z80</td>
<td>ZSDCC (Z88DK)</td>
<td><code>--opt-code-size</code></td>
<td>Ottimizza memoria</td>
</tr>
<tr>
<td>Zilog Z80</td>
<td>SCCZ80 (Z88DK)</td>
<td><code>--opt-code-speed</code></td>
<td>Ottimizza velocità</td>
</tr>
<tr>
<td>MOS 6502</td>
<td>CC65</td>
<td><code>-Oi</code>, <code>-Os</code></td>
<td>Ottimizza velocità</td>
</tr>
</tbody>
</table><p><strong>Problemi noti</strong></p>
<ul>
<li>CC65: <code>-Cl</code> impedisce la ricorsione</li>
<li>CMOC: <code>-O2</code> ha dei bug</li>
<li>ZSDCC: ha dei bug a prescindere dalle opzioni e ne ha altri presenti con <code>-SO3</code> in assenza di <code>--max-alloc-node20000</code>.</li>
</ul>
<h3 id="ottimizzazione-più-sicura">Ottimizzazione più sicura</h3>
<p>Per ovviare a i problemi sopramenzionati e ridurre i tempi di compilazione (soprattutto per l’architettura Z80) si consiglia:</p>

<table>
<thead>
<tr>
<th>Architettura</th>
<th>Compilatore</th>
<th>Opzioni</th>
</tr>
</thead>
<tbody>
<tr>
<td>Zilog Z80</td>
<td>SCCZ80 (Z88DK)</td>
<td><code>-O3</code></td>
</tr>
<tr>
<td>MOS 6502</td>
<td>CC65</td>
<td><code>-O</code></td>
</tr>
<tr>
<td>Motorola 6809</td>
<td>CMOC</td>
<td><code>-O1</code></td>
</tr>
</tbody>
</table><h2 id="evitare-il-linking-di-codice-inutile">Evitare il linking di codice inutile</h2>
<p>I compilatori che trattiamo non sempre saranno capaci di eliminare il codice non usato. Dobbiamo quindi evitare di includere codice non utile per essere sicuri che non finisca nel binario prodotto.</p>
<p>Possiamo fare ancora meglio con alcuni dei nostri compilatori, istruendoli a non includere alcune librerie standard o persino alcune loro parti se siamo sicuri di non doverle usare.</p>
<h3 id="evitare-la-standard-lib">Evitare la standard lib</h3>
<p>Evitare nel proprio codice la libraria standard nei casi in cui avrebbe senso, può ridurre la taglia del codice in maniera considerevole.</p>
<h4 id="cpm-80-solo-getchar-e-putcharc">[cp/m-80] Solo <em>getchar()</em> e <em>putchar(c)</em></h4>
<p>Questa regola è generale ma è particolarmente valida quando si usa ACK per produrre un binario per CP/M-80. In questo caso consiglio di usare esclusivamente <code>getchar()</code> e <code>putchar(c)</code> e implementare tutto il resto.</p>
<h4 id="z88dk-pragmas-per-non-generare-codice">[z88dk] Pragmas per non generare codice</h4>
<p>Z88DK mette a disposizione una serie di <em>pragma</em> per istruire il compilatore a non generare codice inutile.</p>
<p>Per esempio:</p>
<pre><code>#pragma printf = "%c %u"
</code></pre>
<p>includerà solo i convertitori per <code>%c</code> e <code>%u</code> escludendo tutto il codice per gli altri.</p>
<pre><code>#pragma-define:CRT_INITIALIZE_BSS=0
</code></pre>
<p>non genera codice per l’inizializzazione dell’area di memoria BSS.</p>
<pre><code>#pragma output CRT_ON_EXIT = 0x10001
</code></pre>
<p>il programma non fa nulla alla sua uscita (non gestisce il ritorno al BASIC)</p>
<pre><code>#pragma output CLIB_MALLOC_HEAP_SIZE = 0
</code></pre>
<p>elimina lo heap della memoria dinamica (nessuna malloc possibile)</p>
<pre><code>#pragma output CLIB_STDIO_HEAP_SIZE = 0
</code></pre>
<p>elimina lo heap di stdio (non gestisce l’apertura di file)</p>
<p>Alcuni esempi sono in<br>
<a href="https://github.com/Fabrizio-Caruso/CROSS-CHASE/blob/master/src/cross_lib/cfg/z88dk">https://github.com/Fabrizio-Caruso/CROSS-CHASE/blob/master/src/cross_lib/cfg/z88dk</a></p>
<h2 id="usare-le-routine-presenti-in-rom">Usare le routine presenti in ROM</h2>
<p>La stragrande maggioranza dei sistemi 8-bit (quasi tutti i computer) prevede svariate routine nelle ROM. E’ quindi importante conoscerle per usarle. Per usarle esplicitamente dovremo scrivere del codice Assembly da richiamare da C. Il modo d’uso dell’Assembly assieme al C può avvenire in modo <em>in line</em> (codice Assembly integrato all’interno di funzioni C) oppure con file separati da linkare al C ed è diverso in ogni dev-kit. Per i dettagli consigliamo di leggere i manuali dei vari dev-kit.</p>
<p>Questo è molto importante per i sistemi che non sono (ancora) supportati dai compilatori e per i quali bisogna scrivere da zero tutte le routine per l’input/output.</p>
<p>Esempio (preso da <a href="https://github.com/Fabrizio-Caruso/CROSS-CHASE/blob/master/src/cross_lib/display/display_macros.c">https://github.com/Fabrizio-Caruso/CROSS-CHASE/blob/master/src/cross_lib/display/display_macros.c</a>)</p>
<p>Per il display di caratteri sullo schermo per i Thomson Mo5, Mo6 e Olivetti Prodest PC128 (sistemi non supportati da nessun compilatore) piuttosto che scrivere una routine da zero possiamo affidarci ad una routine Assembly presente nella ROM:</p>
<pre><code>	void PUTCH(unsigned char ch)
	{
		asm
		{
			ldb ch
			swi
			.byte 2
		}
	}
</code></pre>
<h4 id="le-librerie-spesso-lo-fanno-per-noi">Le librerie spesso lo fanno per noi</h4>
<p>Fortunatamente spesso potremo usare le routine della ROM implicitamente senza fare alcuna fatica perché le librerie di supporto ai target dei nostri dev-kit, lo fanno già per noi. Usare una routine della ROM ci fa risparmiare codice ma può imporci dei vincoli perché per esempio potrebbero non fare esattamente quello che vogliamo oppure usano alcune aree della RAM (buffer) che noi potremmo volere usare in modo diverso.</p>
<h2 id="sfruttare-lhardware-specifico">Sfruttare l’hardware specifico</h2>
<p>Come visto nelle sezioni precedenti, anche se programmiamo in C non dobbiamo dimenticare l’hardware specifico per il quale stiamo scrivendo del codice.<br>
In alcuni casi conoscere l’hardware può aiutarci a scrivere codice molto più compatto e/o più veloce.</p>
<h3 id="usare-le-estensioni-ascii-specifiche">Usare le estensioni ASCII specifiche</h3>
<p>Per esempio, è inutile ridefinire dei caratteri per fare della grafica se il sistema dispone già di caratteri utili al nostro scopo sfruttando l’estensione specifica dei caratteri ASCII (ATASCII, PETSCII, SHARPSCII, etc.).</p>
<h3 id="sfruttare-i-chip-grafici">Sfruttare i chip grafici</h3>
<p>Conoscere il chip grafico può aiutarci a risparmiare tanta ram.</p>
<p>Esempio (Chip della serie VDP tra cui il TMS9918A presente su MSX, Spectravideo, Memotech MTX, Sord M5, etc.)<br>
I sistemi basati su questo chip prevedono una modalità video testuale (<em>Mode 1</em>)  in cui il colore del carattere è implicitamente dato dal codice del carattere. Se usiamo questo speciale modo video, sarà quindi sufficiente un singolo byte per definire il carattere ed il suo colore con un notevole risparmio in termini di memoria.</p>
<p>Esempio (Commodore Vic 20)<br>
Il Commodore Vic 20 è un caso veramente speciale perché prevede dei limiti hardware (RAM totale: 5k, RAM disponibile per il codice: 3,5K) ma anche dei trucchi per superarli almeno in parte:</p>
<ul>
<li>In realtà dispone anche di 1024 nibble di RAM aggiuntiva speciale per gli attributi colore</li>
<li>Pur avendo soltanto 3,5k di memoria RAM contigua per il codice, molta altra RAM è facilmente sfruttabile per dati (buffer cassetta, buffer comando INPUT del BASIC)</li>
<li>La caratteristica più sorprendente è che il chip grafico VIC può mappare una parte dei caratteri in RAM lasciandone metà definiti dalla ROM</li>
</ul>
<p>Quindi, sfruttiamo implicitamente la prima caratteristica accedendo ai colori senza usare la RAM comune.<br>
Possiamo mappare le nostre variabili nei vari buffer non utilizzati.<br>
Se ci bastano n (n&lt;=64) caratteri ridefiniti possiamo mapparne solo 64 con <code>POKE(0x9005,0xFF);</code> Ne potremo usare anche meno di 64 lasciando il resto per il codice ma mantenendo in aggiunta 64 caratteri standard senza alcun dispendio di memoria per i caratteri standard.</p>

