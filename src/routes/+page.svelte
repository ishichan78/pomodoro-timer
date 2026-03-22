<script lang="ts">
  // --- 状態管理 ---
  type Mode = 'work' | 'shortBreak' | 'longBreak';

  const TIMES: Record<Mode, number> = {
    work: 25 * 60,
    shortBreak: 5 * 60,
    longBreak: 15 * 60,
  };

  const LABELS: Record<Mode, string> = {
    work: '作業',
    shortBreak: '短い休憩',
    longBreak: '長い休憩',
  };

  let mode: Mode = $state('work');
  let timeLeft: number = $state(TIMES['work']);
  let isRunning: boolean = $state(false);
  let pomodoroCount: number = $state(0);
  let intervalId: ReturnType<typeof setInterval> | null = null;

  // --- 表示用：MM:SS形式に変換 ---
  function formatTime(seconds: number): string {
    const m = Math.floor(seconds / 60).toString().padStart(2, '0');
    const s = (seconds % 60).toString().padStart(2, '0');
    return `${m}:${s}`;
  }

  // --- モード切り替え ---
  function setMode(newMode: Mode) {
    stopTimer();
    mode = newMode;
    timeLeft = TIMES[newMode];
  }

  // --- タイマー開始 ---
  function startTimer() {
    if (isRunning) return;
    isRunning = true;
    intervalId = setInterval(() => {
      if (timeLeft > 0) {
        timeLeft -= 1;
      } else {
        // タイマー終了
        stopTimer();
        handleTimerEnd();
      }
    }, 1000);
  }

  // --- タイマー停止 ---
  function stopTimer() {
    if (intervalId) {
      clearInterval(intervalId);
      intervalId = null;
    }
    isRunning = false;
  }

  // --- リセット ---
  function resetTimer() {
    stopTimer();
    timeLeft = TIMES[mode];
  }

  // --- タイマー終了時の処理 ---
  function handleTimerEnd() {
    if (mode === 'work') {
      pomodoroCount += 1;
      // 4回ごとに長い休憩
      if (pomodoroCount % 4 === 0) {
        setMode('longBreak');
      } else {
        setMode('shortBreak');
      }
    } else {
      setMode('work');
    }
  }

  // --- 進捗率（円グラフ用）---
  let progress = $derived(1 - timeLeft / TIMES[mode]);
  let circumference = 2 * Math.PI * 90; // 半径90の円
  let dashOffset = $derived(circumference * (1 - progress));
</script>

<!-- タイトル -->
<svelte:head>
  <title>ポモドーロタイマー</title>
</svelte:head>

<main>
  <!-- モード選択タブ -->
  <div class="tabs">
    {#each (['work', 'shortBreak', 'longBreak'] as Mode[]) as m}
      <button
        class="tab"
        class:active={mode === m}
        onclick={() => setMode(m)}
      >
        {LABELS[m]}
      </button>
    {/each}
  </div>

  <!-- 円形タイマー -->
  <div class="timer-container">
    <svg width="220" height="220" viewBox="0 0 220 220">
      <!-- 背景の円 -->
      <circle cx="110" cy="110" r="90" fill="none" stroke="#e0e0e0" stroke-width="10" />
      <!-- 進捗の円 -->
      <circle
        cx="110" cy="110" r="90"
        fill="none"
        stroke={mode === 'work' ? '#e74c3c' : '#2ecc71'}
        stroke-width="10"
        stroke-linecap="round"
        stroke-dasharray={circumference}
        stroke-dashoffset={dashOffset}
        transform="rotate(-90 110 110)"
        style="transition: stroke-dashoffset 0.5s ease;"
      />
      <!-- 時間表示 -->
      <text x="110" y="118" text-anchor="middle" class="timer-text">
        {formatTime(timeLeft)}
      </text>
    </svg>
  </div>

  <!-- ポモドーロ数 -->
  <p class="count">🍅 × {pomodoroCount}</p>

  <!-- コントロールボタン -->
  <div class="controls">
    {#if !isRunning}
      <button class="btn start" onclick={startTimer}>▶ スタート</button>
    {:else}
      <button class="btn pause" onclick={stopTimer}>⏸ 一時停止</button>
    {/if}
    <button class="btn reset" onclick={resetTimer}>↺ リセット</button>
  </div>
</main>

<style>
  main {
    display: flex;
    flex-direction: column;
    align-items: center;
    justify-content: center;
    height: 100vh;
    font-family: 'Helvetica Neue', Arial, sans-serif;
    background: #1a1a2e;
    color: #eee;
    gap: 1.5rem;
  }

  .tabs {
    display: flex;
    gap: 0.5rem;
    background: #16213e;
    padding: 0.4rem;
    border-radius: 999px;
  }

  .tab {
    padding: 0.4rem 1.2rem;
    border: none;
    border-radius: 999px;
    background: transparent;
    color: #aaa;
    cursor: pointer;
    font-size: 0.9rem;
    transition: all 0.2s;
  }

  .tab.active {
    background: #e74c3c;
    color: white;
    font-weight: bold;
  }

  .timer-container {
    position: relative;
  }

  .timer-text {
    font-size: 2.8rem;
    font-weight: bold;
    fill: white;
    font-family: 'Helvetica Neue', Arial, sans-serif;
  }

  .count {
    font-size: 1.4rem;
    margin: 0;
  }

  .controls {
    display: flex;
    gap: 1rem;
  }

  .btn {
    padding: 0.7rem 2rem;
    border: none;
    border-radius: 999px;
    font-size: 1rem;
    cursor: pointer;
    font-weight: bold;
    transition: transform 0.1s, opacity 0.2s;
  }

  .btn:active {
    transform: scale(0.96);
  }

  .start {
    background: #e74c3c;
    color: white;
  }

  .pause {
    background: #f39c12;
    color: white;
  }

  .reset {
    background: #2c3e50;
    color: white;
  }
</style>