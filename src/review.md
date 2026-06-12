# Code review per `app.vue`

**Data:** 12 giugno 2026 <br>
**Percorso:** `./App.vue`<br>
**Reviewer:** Abdel<br>

## Sintesi generale

L'app presenta una buona struttura di base. Tuttavia, ci sono diverse criticità legate all'uso della reattività (approccio troppo imperativo), problemi di performance nel template e pratiche di UX sconsigliate.

Di seguito i dettagli delle problematiche riscontrate e i suggerimenti per la risoluzione.

---

## Dettaglio della reviw

**[Riga 6][Riga 88] <br>Props senza default:** Il componente accetta `title`, ma non definisce un comportamento di fallback. visto che il parent non passa la prop, il tag `<h1>` risulta vuoto.

**[Riga 12] <br>Variabile `filteredUsers`:** Dichiarata con `let` invece di `const`.
La ref non cambia nel tempo, si modifica il suo campo (`.value`)
percui serve dicchiararle sempre con `const`.

**[Riga 14] <br>Inizializzazione di `isLoading`:** La variabile `isLoading` è inizializzata a `false` ma la chiamata API parte subito. Dovrebbe partire come `true` per mostrare immediatamente lo stato di caricamento corretto.

**[Riga 19] <br>Codice morto:** Lo stato `formState` è dichiarato tramite `reactive` ma non è mai utilizzato in tutto il componente. Va rimosso.

**[Riga 27] <br>API Fetch:** la fetch è inserita direttamente nel blocco setup senza essere avvolta in un lifecycle hook.

<ul>
<small> <li> <i>Suggerimento:</i> Spostare la chiamata `fetch` all'interno di `onMounted()` per garantire che venga eseguita solo quando il componente è effettivamente inserito nel DOM, evitando potenziali comportamenti anomali o side-effects in fase di caricamento.</small>
</ul>

**[Righe 45-51] <br>Filtraggio:** Il filtraggio viene gestito in modo imperativo aggiornando manualmente `filteredUsers` tramite una funzione scatenata da `@input`. Questo è molto inefficiente ed è soggetto a bug.

<ul>
<small> <li> <i>Suggerimento:</i>  Sostituire la funzione `applyFilter` e lo stato `filteredUsers` con una **`computed` property**. Questo rende il codice dichiarativo, gestisce automaticamente la cache e si aggiorna solo quando `users` o `searchFilter` cambiano.</small>
</ul>

**[Riga 56] <br>Filtro Case-Sensitive:** `users.value[i].name.includes(...)` è case-sensitive. c'è distenzione tra "Marco" e "marco".
_Suggerimento:_ bisogna aggiungere un .toLowerCase (oppure to upper) ad entrambe le stringhe da confrontare

**[Righe 78] <br>Alert:** L'utilizzo di `watch` per scatenare un `alert()` del browser ogni volta che cambia il conteggio blocca l'esecuzione del thread principale e offre un'esperienza utente estremamente povera.

<ul>
<small> <li> <i>Suggerimento:</i>  Rimuovere il watch e l'alert. La UI mostra già il conteggio aggiornato (`{{ favoriteCount }}`).</small> </ul>

**[Riga 105] <br>Mancanza di `:key` nel `v-for`:**
Iterando su un array bisogna fornire una `:key` univoca per ogni elemento, in modo che Vue lo possa tracciare correttamente.

<ul>
<small> <li> <i>Suggerimento:</i>  Aggiungere `:key="user.id"` (assumendo che i dati abbiano un campo `id` univoco). </small> </ul>

**[Riga 122] <br> L'app non è responsive**
l'applicazione è ottimizzata solo per un utilizzo su schermi medio-grandi (tablet o pc), il codice CSS non gestisce l'adattamento dell'applicazione a schermi diversi

<ul>
<small> <li> <i>Soluzione:</i>  utilizzare i media queries </small> </ul>
