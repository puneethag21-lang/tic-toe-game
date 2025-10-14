# tic-tac-toe-game
I created a responsive Tic-Tac-Toe game using HTML, CSS, and JavaScript! and with the help of ai  It works smoothly on both mobile and desktop, with 2-player and AI (computer) modes.  This project helped me learn about game logic, DOM manipulation, and responsive 
<!doctype html>
<html lang="en">
<head>
  <meta charset="utf-8" />
  <meta name="viewport" content="width=device-width, initial-scale=1" />
  <title>Tic‑Tac‑Toe — Responsive</title>
  <style>
    :root{
      --bg:#0f1724; --card:#0b1220; --accent:#06b6d4; --muted:#94a3b8; --win:#10b981;
      --cell-size: calc(min(72vmin, 70vh) / 3);
      font-family: Inter, system-ui, -apple-system, "Segoe UI", Roboto, "Helvetica Neue", Arial;
    }
    *{box-sizing:border-box}
    html,body{height:100%}
    body{
      margin:0; background:linear-gradient(180deg,var(--bg),#071027); color:#e6eef8; display:flex; align-items:center; justify-content:center; padding:20px;
    }
    .wrap{width:100%; max-width:900px}
    .card{
      background:linear-gradient(180deg, rgba(255,255,255,0.02), rgba(255,255,255,0.01));
      border-radius:16px; padding:18px; box-shadow:0 8px 30px rgba(2,6,23,0.7);
    }
    header{display:flex; gap:12px; align-items:center; justify-content:space-between; margin-bottom:12px}
    .title{display:flex; gap:12px; align-items:center}
    .logo{width:48px;height:48px;border-radius:10px;background:linear-gradient(135deg,var(--accent),#7c3aed); display:flex;align-items:center;justify-content:center;font-weight:700}
    h1{font-size:1.05rem;margin:0}
    p.lead{margin:0;color:var(--muted);font-size:0.87rem}

    .controls{display:flex; gap:8px; align-items:center}
    select, button{background:transparent;border:1px solid rgba(255,255,255,0.06); padding:8px 10px; color:inherit; border-radius:10px}
    button.primary{background:var(--accent); color:#042024; border:none}

    .board-wrap{display:flex; gap:18px; align-items:flex-start; flex-wrap:wrap}

    .board{
      width:calc(var(--cell-size) * 3);
      height:calc(var(--cell-size) * 3);
      display:grid; grid-template-columns:repeat(3,1fr); grid-template-rows:repeat(3,1fr); gap:6px; touch-action: manipulation
    }
    .cell{
      background:linear-gradient(180deg, rgba(255,255,255,0.01), rgba(255,255,255,0.02));
      border-radius:12px; display:flex; align-items:center; justify-content:center; font-size:calc(var(--cell-size) * 0.28); user-select:none; -webkit-user-select:none; cursor:pointer; transition:transform .12s ease, background .12s ease;
    }
    .cell:active{transform:scale(.98)}
    .cell.x{color:#f97316}
    .cell.o{color:#60a5fa}

    .info{flex:1; min-width:220px}
    .status{background:rgba(255,255,255,0.02); padding:12px; border-radius:10px; margin-bottom:10px}
    .score{display:flex; gap:10px; margin-bottom:10px}
    .score .item{background:rgba(255,255,255,0.02); padding:8px 10px; border-radius:8px}

    .small{font-size:0.85rem; color:var(--muted)}

    footer{display:flex; gap:8px; justify-content:flex-end; margin-top:12px}

    /* winner highlight */
    .cell.win{background:linear-gradient(90deg, rgba(16,185,129,0.12), rgba(6,182,212,0.06)); box-shadow:0 6px 18px rgba(2,6,23,0.6) inset}

    /* responsive tweaks */
    @media (max-width:640px){
      .board-wrap{flex-direction:column; align-items:center}
      .info{width:100%}
      :root{--cell-size:calc(min(92vmin, 60vh) / 3)}
      header{flex-direction:column; align-items:flex-start; gap:8px}
      footer{justify-content:space-between}
    }

    /* small accessibility helpers */
    .sr-only{position:absolute;width:1px;height:1px;padding:0;margin:-1px;overflow:hidden;clip:rect(0,0,0,0);white-space:nowrap;border:0}
  </style>
</head>
<body>
  <div class="wrap">
    <div class="card">
      <header>
        <div class="title">
          <div class="logo">TTT</div>
          <div>
            <h1>Tic‑Tac‑Toe</h1>
            <p class="lead">Play with a friend or the computer — responsive and mobile friendly.</p>
          </div>
        </div>

        <div class="controls">
          <label class="small">Mode
            <select id="mode">
              <option value="pvp">2 players (Local)</option>
              <option value="pvc">Vs Computer</option>
            </select>
          </label>
          <label class="small">Starter
            <select id="starter">
              <option value="x">X (you)</option>
              <option value="o">O</option>
            </select>
          </label>
          <button id="reset" title="Restart game">Restart</button>
        </div>
      </header>

      <div class="board-wrap">
        <div class="board" id="board" role="grid" aria-label="Tic tac toe board">
          <!-- 9 cells inserted by JS -->
        </div>

        <aside class="info">
          <div class="status" id="status">Turn: <strong id="turn">X</strong></div>
          <div class="score" aria-live="polite">
            <div class="item">X: <span id="scoreX">0</span></div>
            <div class="item">O: <span id="scoreO">0</span></div>
            <div class="item">Draws: <span id="scoreD">0</span></div>
          </div>
          <div class="small">Tip: tap cells to play. On small screens the layout stacks vertically.</div>
          <footer>
            <button id="undo" title="Undo last move">Undo</button>
            <button id="newRound" class="primary">New Round</button>
          </footer>
        </aside>
      </div>
    </div>
  </div>

  <script>
    // Basic tic-tac-toe with optional computer opponent (minimax).
    (function(){
      const boardEl = document.getElementById('board');
      const statusEl = document.getElementById('status');
      const turnEl = document.getElementById('turn');
      const modeEl = document.getElementById('mode');
      const starterEl = document.getElementById('starter');
      const resetBtn = document.getElementById('reset');
      const newRoundBtn = document.getElementById('newRound');
      const undoBtn = document.getElementById('undo');
      const scoreX = document.getElementById('scoreX');
      const scoreO = document.getElementById('scoreO');
      const scoreD = document.getElementById('scoreD');

      let state = Array(9).fill(null); // 'x','o' or null
      let current = 'x';
      let scores = {x:0,o:0,d:0};
      let history = [];
      let gameOver = false;

      const winLines = [
        [0,1,2],[3,4,5],[6,7,8],
        [0,3,6],[1,4,7],[2,5,8],
        [0,4,8],[2,4,6]
      ];

      function renderBoard(){
        boardEl.innerHTML = '';
        state.forEach((v,i)=>{
          const cell = document.createElement('button');
          cell.className = 'cell';
          cell.setAttribute('role','gridcell');
          cell.setAttribute('aria-label', v ? (v.toUpperCase()+' at '+(i+1)) : 'empty');
          cell.dataset.index = i;
          cell.tabIndex = 0;
          if(v){ cell.classList.add(v); cell.textContent = v.toUpperCase(); }
          cell.addEventListener('click', onCellClick);
          boardEl.appendChild(cell);
        });
      }

      function setStatus(text){ statusEl.querySelector('strong') ? turnEl.textContent = text : null }

      function onCellClick(e){
        const i = Number(e.currentTarget.dataset.index);
        if(gameOver) return;
        if(state[i]) return; // occupied
        makeMove(i, current);

        // if vs computer and not over, trigger CPU after a tiny delay
        if(!gameOver && modeEl.value === 'pvc'){
          const cpu = (starterEl.value === current) ? 'o' : 'o'; // CPU plays O by default when pvc
          // If player chose to start as O, CPU should be X - but for simplicity, CPU will always be the opposite of current turn now.
          if(!gameOver) setTimeout(()=> cpuPlay(), 220);
        }
      }

      function makeMove(index, player){
        state[index] = player;
        history.push({index,player});
        renderBoard();
        const result = checkWinner(state);
        if(result){
          gameOver = true;
          if(result.winner){
            highlight(result.line);
            scores[result.winner]++;
            updateScores();
            setStatus(result.winner.toUpperCase() + ' wins!');
          } else {
            scores.d++;
            updateScores();
            setStatus('Draw');
          }
        } else {
          current = current === 'x' ? 'o' : 'x';
          setStatus(current.toUpperCase());
        }
      }

      function cpuPlay(){
        // CPU plays as the opposite of the last player (simple minimax)
        const cpu = current; // CPU moves when it's its turn
        // find best move
        const best = bestMove(state.slice(), cpu);
        if(best !== null){
          makeMove(best, cpu);
        }
      }

      function bestMove(boardArr, player){
        // Minimax returns index for best move or null if none
        const avail = boardArr.map((v,i)=> v ? null : i).filter(i=>i!==null);
        if(avail.length === 9){ // take center or corner quickly
          return 4; // center
        }
        const opponent = player === 'x' ? 'o' : 'x';

        // minimax implementation
        function minimax(b, turn){
          const res = checkWinner(b);
          if(res){
            if(res.winner === player) return {score: 10};
            else if(res.winner === opponent) return {score: -10};
            else return {score: 0};
          }

          const moves = [];
          for(let i=0;i<9;i++){
            if(!b[i]){
              b[i] = turn;
              const outcome = minimax(b, turn===player ? opponent : player);
              moves.push({index:i, score: outcome.score});
              b[i] = null;
            }
          }

          // choose best
          let bestMove = null;
          if(turn === player){ // maximize
            let bestScore = -Infinity;
            moves.forEach(m => { if(m.score > bestScore){ bestScore = m.score; bestMove = m; } });
          } else { // minimize
            let bestScore = Infinity;
            moves.forEach(m => { if(m.score < bestScore){ bestScore = m.score; bestMove = m; } });
          }
          return bestMove || {score:0};
        }

        const choice = minimax(boardArr, player);
        return choice && typeof choice.index === 'number' ? choice.index : null;
      }

      function checkWinner(b){
        for(const line of winLines){
          const [a,b1,c] = line;
          if(b[a] && b[a] === b[b1] && b[a] === b[c]){
            return {winner: b[a], line};
          }
        }
        if(b.every(Boolean)) return {winner: null};
        return null;
      }

      function highlight(line){
        if(!line) return;
        line.forEach(i=>{
          const el = boardEl.querySelector('[data-index="'+i+'"]');
          if(el) el.classList.add('win');
        });
      }

      function updateScores(){ scoreX.textContent = scores.x; scoreO.textContent = scores.o; scoreD.textContent = scores.d }

      function resetBoard(){ state = Array(9).fill(null); history = []; gameOver=false; current = starterEl.value || 'x'; renderBoard(); setStatus(current.toUpperCase()); removeWinHighlights(); }
      function newRound(){ resetBoard(); }
      function removeWinHighlights(){ boardEl.querySelectorAll('.cell.win').forEach(c=>c.classList.remove('win')) }

      resetBtn.addEventListener('click', ()=>{ scores={x:0,o:0,d:0}; updateScores(); newRound(); });
      newRoundBtn.addEventListener('click', newRound);
      modeEl.addEventListener('change', ()=>{ newRound(); });
      starterEl.addEventListener('change', ()=>{ current = starterEl.value; newRound(); });

      undoBtn.addEventListener('click', ()=>{
        if(history.length===0 || gameOver) return;
        const last = history.pop();
        state[last.index] = null;
        current = last.player; // restore turn to the player who made the undone move
        gameOver = false;
        removeWinHighlights();
        renderBoard();
        setStatus(current.toUpperCase());
      });

      // initialize
      (function init(){
        // create 9 cells (renderBoard will do this too)
        resetBoard();
        updateScores();

        // keyboard support: space/enter on focused cell
        boardEl.addEventListener('keydown', (e)=>{
          if(e.key === 'Enter' || e.key === ' '){
            e.preventDefault();
            const target = document.activeElement;
            if(target && target.classList.contains('cell')) target.click();
          }
        });

      })();

    })();
  </script>
</body>
</html>
