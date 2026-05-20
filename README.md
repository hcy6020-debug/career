[index.html](https://github.com/user-attachments/files/28043860/index.html)
import { useState, useEffect, useRef, useCallback } from "react";

const QUESTIONS = [
  { q: "사물, 기계, 도구를 다루고 몸으로 문제를 해결하는 유형은?", answer: "R", choices: ["R","I","A","S"] },
  { q: "수학·과학 현상을 관찰하고 논리적·합리적 사고를 즐기는 유형은?", answer: "I", choices: ["I","E","C","R"] },
  { q: "비구조적 환경에서 창작 활동에 몰두하고 감수성이 풍부한 유형은?", answer: "A", choices: ["A","S","I","E"] },
  { q: "다른 사람을 가르치고 돌보며 봉사 정신이 강한 유형은?", answer: "S", choices: ["S","R","E","C"] },
  { q: "조직적 목표를 위해 다른 사람을 이끌고 설득하는 유형은?", answer: "E", choices: ["E","A","S","I"] },
  { q: "수를 다루고 자료를 정리하며 꼼꼼하고 계획적인 유형은?", answer: "C", choices: ["C","R","I","A"] },
  { q: "과학자, 의사, 생물학자가 속하는 Holland 유형은?", answer: "I", choices: ["I","R","A","C"] },
  { q: "예술가, 디자이너, 음악가가 속하는 Holland 유형은?", answer: "A", choices: ["A","E","S","I"] },
  { q: "사회복지사, 교육자, 간호사가 속하는 Holland 유형은?", answer: "S", choices: ["S","C","R","A"] },
  { q: "공인회계사, 세무사, 은행원이 속하는 Holland 유형은?", answer: "C", choices: ["C","E","S","R"] },
  { q: "기업경영인, 정치가, 외교관이 속하는 Holland 유형은?", answer: "E", choices: ["E","I","A","C"] },
  { q: "자동차 정비원, 항공기 조종사, 운동선수가 속하는 유형은?", answer: "R", choices: ["R","S","E","I"] },
  { q: "혼자서 독립적으로 일하고 집중력이 강한 유형은?", answer: "I", choices: ["I","C","S","E"] },
  { q: "여가로 운동, 캠핑, 탐험 같은 야외활동을 즐기는 유형은?", answer: "R", choices: ["R","A","E","S"] },
  { q: "여가로 음악, 미술, 영화, 댄스 창작 활동을 즐기는 유형은?", answer: "A", choices: ["A","I","C","R"] },
  { q: "여가로 퍼즐, 낱말 맞추기, 정리정돈을 즐기는 유형은?", answer: "C", choices: ["C","S","I","E"] },
  { q: "여가로 자원봉사, 친목 활동을 즐기는 유형은?", answer: "S", choices: ["S","E","R","A"] },
  { q: "여가로 가상체험, 승부가 걸린 활동을 즐기는 유형은?", answer: "E", choices: ["E","C","A","S"] },
  { q: "보상에 민감하고 모험심이 강하며 리더 역할을 좋아하는 유형은?", answer: "E", choices: ["E","R","A","I"] },
  { q: "맡은 일에 끝까지 책임지고 약속을 잘 지키는 유형은?", answer: "C", choices: ["C","S","E","R"] },
];

const TYPE_INFO = {
  R: { name: "실재형", color: "#e74c3c", emoji: "⚔️" },
  I: { name: "탐구형", color: "#2980b9", emoji: "🔬" },
  A: { name: "예술형", color: "#8e44ad", emoji: "🎨" },
  S: { name: "사회형", color: "#27ae60", emoji: "💚" },
  E: { name: "기업형", color: "#e67e22", emoji: "👑" },
  C: { name: "관습형", color: "#7f8c8d", emoji: "📋" },
};

const W = 640, H = 380;
const BALLOON_COLORS = ["#e74c3c","#2980b9","#8e44ad","#27ae60","#e67e22","#7f8c8d","#f39c12","#1abc9c"];

function shuffle(arr) { return [...arr].sort(() => Math.random() - 0.5); }

function makeBalloon(label, isAnswer, id) {
  return {
    id, label, isAnswer,
    x: 60 + Math.random() * (W - 120),
    y: H + 60 + Math.random() * 80,
    vx: (Math.random() - 0.5) * 1.2,
    vy: -(1.4 + Math.random() * 1.0),
    r: 36,
    color: BALLOON_COLORS[Math.floor(Math.random() * BALLOON_COLORS.length)],
    pop: false,
    popAnim: 0,
    wobble: Math.random() * Math.PI * 2,
  };
}

export default function Game() {
  const [screen, setScreen] = useState("intro");
  const [score, setScore] = useState(0);
  const [timeLeft, setTimeLeft] = useState(60);
  const [correct, setCorrect] = useState(0);
  const [wrong, setWrong] = useState(0);
  const [qIdx, setQIdx] = useState(0);
  const [questions] = useState(() => shuffle(QUESTIONS));
  const [flash, setFlash] = useState(null); // "correct" | "wrong"

  const canvasRef = useRef(null);
  const stateRef = useRef(null);
  const rafRef = useRef(null);
  const idRef = useRef(0);

  const startGame = useCallback(() => {
    setScreen("game");
    setScore(0);
    setTimeLeft(60);
    setCorrect(0);
    setWrong(0);
    setQIdx(0);
    setFlash(null);
  }, []);

  useEffect(() => {
    if (screen !== "game") return;
    const canvas = canvasRef.current;
    if (!canvas) return;
    const ctx = canvas.getContext("2d");

    const q = questions[0 % questions.length];
    const balloons = q.choices.map(c => makeBalloon(c, c === q.answer, idRef.current++));

    const state = {
      balloons,
      particles: [],
      score: 0,
      timeLeft: 60,
      correct: 0,
      wrong: 0,
      qIdx: 0,
      over: false,
      flashTimer: 0,
      flashColor: null,
    };
    stateRef.current = state;

    const loadQuestion = (idx) => {
      const nq = questions[idx % questions.length];
      state.balloons = nq.choices.map(c => makeBalloon(c, c === nq.answer, idRef.current++));
    };

    const handleClick = (e) => {
      const s = stateRef.current;
      if (s.over) return;
      const rect = canvas.getBoundingClientRect();
      const scaleX = W / rect.width;
      const scaleY = H / rect.height;
      const mx = (e.clientX - rect.left) * scaleX;
      const my = (e.clientY - rect.top) * scaleY;
      let hit = false;
      s.balloons.forEach(b => {
        if (b.pop) return;
        const dx = mx - b.x, dy = my - b.y;
        if (Math.sqrt(dx*dx+dy*dy) < b.r && !hit) {
          hit = true;
          b.pop = true;
          for (let i = 0; i < 12; i++) s.particles.push({
            x: b.x, y: b.y,
            dx: (Math.random()-0.5)*5, dy: (Math.random()-0.5)*5,
            life: 30, color: b.color
          });
          if (b.isAnswer) {
            s.score += 10;
            s.correct += 1;
            s.flashTimer = 30;
            s.flashColor = "correct";
            setCorrect(c => c + 1);
          } else {
            s.score = Math.max(0, s.score - 5);
            s.wrong += 1;
            s.flashTimer = 30;
            s.flashColor = "wrong";
            setWrong(c => c + 1);
          }
          setScore(s.score);
          setTimeout(() => {
            s.qIdx = (s.qIdx + 1) % questions.length;
            loadQuestion(s.qIdx);
            setQIdx(s.qIdx);
          }, 400);
        }
      });
    };

    canvas.addEventListener("click", handleClick);

    let lastTime = 0;
    const loop = (ts) => {
      const dt = Math.min((ts - lastTime) / 16, 3);
      lastTime = ts;
      const s = stateRef.current;

      s.timeLeft -= dt / 60;
      if (s.timeLeft <= 0) { s.timeLeft = 0; s.over = true; }
      setTimeLeft(Math.ceil(s.timeLeft));

      if (s.flashTimer > 0) { s.flashTimer -= dt; setFlash(s.flashTimer > 0 ? s.flashColor : null); }
      else setFlash(null);

      s.balloons.forEach(b => {
        b.wobble += 0.05 * dt;
        if (!b.pop) {
          b.x += b.vx * dt;
          b.y += b.vy * dt;
          if (b.x < b.r || b.x > W - b.r) b.vx *= -1;
          if (b.y < -120) { b.y = H + 60; b.x = 60 + Math.random() * (W - 120); }
        } else {
          b.popAnim += 0.15 * dt;
        }
      });

      s.particles = s.particles.filter(p => p.life > 0);
      s.particles.forEach(p => { p.x += p.dx; p.y += p.dy; p.dy += 0.15; p.life -= dt; });

      ctx.clearRect(0, 0, W, H);
      ctx.fillStyle = s.flashColor === "correct" && s.flashTimer > 0
        ? "rgba(39,174,96,0.08)"
        : s.flashColor === "wrong" && s.flashTimer > 0
        ? "rgba(231,76,60,0.08)"
        : "#f8f4ff";
      ctx.fillRect(0, 0, W, H);

      // 구름 장식
      [[80,60],[300,40],[520,70],[160,120],[440,100]].forEach(([cx,cy]) => {
        ctx.fillStyle = "rgba(255,255,255,0.7)";
        ctx.beginPath(); ctx.arc(cx,cy,28,0,Math.PI*2); ctx.fill();
        ctx.beginPath(); ctx.arc(cx+22,cy+6,20,0,Math.PI*2); ctx.fill();
        ctx.beginPath(); ctx.arc(cx-20,cy+8,18,0,Math.PI*2); ctx.fill();
      });

      s.balloons.forEach(b => {
        if (b.pop && b.popAnim < 1) {
          ctx.save();
          ctx.globalAlpha = 1 - b.popAnim;
          ctx.translate(b.x, b.y);
          ctx.scale(1 + b.popAnim * 0.6, 1 + b.popAnim * 0.6);
          ctx.font = "28px serif";
          ctx.textAlign = "center";
          ctx.fillText("💥", 0, 10);
          ctx.restore();
          return;
        }
        if (b.pop) return;

        const wx = Math.sin(b.wobble) * 3;
        ctx.save();
        ctx.translate(b.x + wx, b.y);

        // 실
        ctx.beginPath();
        ctx.moveTo(0, b.r);
        ctx.quadraticCurveTo(wx * 2, b.r + 22, 0, b.r + 40);
        ctx.strokeStyle = "rgba(0,0,0,0.25)";
        ctx.lineWidth = 1.5;
        ctx.stroke();

        // 풍선 몸체
        ctx.beginPath();
        ctx.ellipse(0, 0, b.r, b.r * 1.15, 0, 0, Math.PI * 2);
        ctx.fillStyle = b.color;
        ctx.fill();

        // 광택
        ctx.beginPath();
        ctx.ellipse(-b.r*0.3, -b.r*0.3, b.r*0.22, b.r*0.15, -Math.PI/4, 0, Math.PI*2);
        ctx.fillStyle = "rgba(255,255,255,0.4)";
        ctx.fill();

        // 꼭지
        ctx.beginPath();
        ctx.moveTo(-4, b.r * 1.15);
        ctx.lineTo(4, b.r * 1.15);
        ctx.lineTo(0, b.r * 1.15 + 8);
        ctx.fillStyle = b.color;
        ctx.fill();

        // 라벨
        ctx.font = "bold 17px sans-serif";
        ctx.textAlign = "center";
        ctx.fillStyle = "#fff";
        ctx.shadowColor = "rgba(0,0,0,0.3)";
        ctx.shadowBlur = 3;
        ctx.fillText(b.label, 0, 6);
        ctx.shadowBlur = 0;

        ctx.restore();
      });

      s.particles.forEach(p => {
        ctx.globalAlpha = p.life / 30;
        ctx.fillStyle = p.color;
        ctx.beginPath();
        ctx.arc(p.x, p.y, 5, 0, Math.PI * 2);
        ctx.fill();
      });
      ctx.globalAlpha = 1;

      if (s.over) {
        setScreen("result");
        return;
      }

      rafRef.current = requestAnimationFrame(loop);
    };

    rafRef.current = requestAnimationFrame(loop);
    return () => {
      cancelAnimationFrame(rafRef.current);
      canvas.removeEventListener("click", handleClick);
    };
  }, [screen, questions]);

  const currentQ = questions[qIdx % questions.length];
  const tColor = timeLeft <= 10 ? "#e74c3c" : timeLeft <= 20 ? "#e67e22" : "#2c3e50";

  if (screen === "intro") return (
    <div style={{ padding: "2rem", textAlign: "center", fontFamily: "var(--font-sans)" }}>
      <div style={{ fontSize: 52, marginBottom: 8 }}>🎈</div>
      <h2 style={{ fontSize: 22, fontWeight: 500, margin: "0 0 8px", color: "var(--color-text-primary)" }}>Holland 유형 풍선 퀴즈</h2>
      <p style={{ color: "var(--color-text-secondary)", marginBottom: 24, fontSize: 15 }}>
        정답 풍선을 터뜨리세요! · 1분 안에 최고 점수 도전
      </p>
      <div style={{ display: "grid", gridTemplateColumns: "repeat(3,1fr)", gap: 10, maxWidth: 420, margin: "0 auto 28px" }}>
        {Object.entries(TYPE_INFO).map(([k, t]) => (
          <div key={k} style={{ background: t.color + "18", border: `1.5px solid ${t.color}55`, borderRadius: 10, padding: "10px 6px", fontSize: 13 }}>
            <div style={{ fontSize: 22 }}>{t.emoji}</div>
            <div style={{ fontWeight: 500, color: "var(--color-text-primary)" }}>{k} · {t.name}</div>
          </div>
        ))}
      </div>
      <div style={{ background: "var(--color-background-secondary)", borderRadius: 10, padding: "12px 20px", maxWidth: 340, margin: "0 auto 24px", fontSize: 13, color: "var(--color-text-secondary)", textAlign: "left", lineHeight: 1.8 }}>
        🎯 정답 풍선 터뜨리기 → <b>+10점</b><br/>
        ❌ 오답 풍선 터뜨리기 → <b>-5점</b><br/>
        ⏱ 제한 시간: <b>1분</b>
      </div>
      <button onClick={startGame} style={{ padding: "12px 40px", fontSize: 16, borderRadius: 10, border: "none", background: "#8e44ad", color: "#fff", cursor: "pointer", fontWeight: 500 }}>
        게임 시작!
      </button>
    </div>
  );

  if (screen === "result") return (
    <div style={{ padding: "2rem", textAlign: "center", fontFamily: "var(--font-sans)" }}>
      <div style={{ fontSize: 48 }}>🎉</div>
      <h2 style={{ fontSize: 22, fontWeight: 500, margin: "8px 0 4px", color: "var(--color-text-primary)" }}>1분 종료!</h2>
      <div style={{ display: "grid", gridTemplateColumns: "repeat(3,1fr)", gap: 10, maxWidth: 360, margin: "20px auto" }}>
        {[["최종 점수", score + "점"], ["정답", correct + "개"], ["오답", wrong + "개"]].map(([l, v]) => (
          <div key={l} style={{ background: "var(--color-background-secondary)", borderRadius: 8, padding: "0.75rem" }}>
            <div style={{ fontSize: 12, color: "var(--color-text-secondary)" }}>{l}</div>
            <div style={{ fontSize: 22, fontWeight: 500, color: "var(--color-text-primary)" }}>{v}</div>
          </div>
        ))}
      </div>
      <div style={{ background: "var(--color-background-secondary)", borderRadius: 12, padding: "1rem 1.5rem", maxWidth: 360, margin: "0 auto 20px", textAlign: "left" }}>
        <div style={{ fontSize: 13, color: "var(--color-text-secondary)", marginBottom: 8 }}>Holland 6가지 유형 요약</div>
        {Object.entries(TYPE_INFO).map(([k, t]) => (
          <div key={k} style={{ display: "flex", alignItems: "center", gap: 8, marginBottom: 4 }}>
            <span style={{ fontSize: 16 }}>{t.emoji}</span>
            <span style={{ fontWeight: 500, fontSize: 13, color: "var(--color-text-primary)", minWidth: 60 }}>{k} {t.name}</span>
          </div>
        ))}
      </div>
      <div style={{ display: "flex", gap: 10, justifyContent: "center" }}>
        <button onClick={startGame} style={{ padding: "8px 24px", borderRadius: 8, border: "none", background: "#8e44ad", color: "#fff", cursor: "pointer", fontSize: 14 }}>다시 도전</button>
        <button onClick={() => setScreen("intro")} style={{ padding: "8px 24px", borderRadius: 8, border: "0.5px solid var(--color-border-secondary)", background: "var(--color-background-primary)", cursor: "pointer", fontSize: 14, color: "var(--color-text-primary)" }}>처음으로</button>
      </div>
    </div>
  );

  return (
    <div style={{ fontFamily: "var(--font-sans)", userSelect: "none" }}>
      <div style={{ display: "flex", justifyContent: "space-between", alignItems: "center", padding: "8px 16px", background: "var(--color-background-secondary)", borderBottom: "0.5px solid var(--color-border-tertiary)" }}>
        <span style={{ fontSize: 14, color: "var(--color-text-secondary)" }}>정답 {correct} · 오답 {wrong}</span>
        <span style={{ fontSize: 20, fontWeight: 500, color: tColor }}>⏱ {timeLeft}초</span>
        <span style={{ fontSize: 14, fontWeight: 500, color: "var(--color-text-primary)" }}>🏆 {score}점</span>
      </div>
      <div style={{
        padding: "10px 16px", textAlign: "center",
        background: flash === "correct" ? "rgba(39,174,96,0.12)" : flash === "wrong" ? "rgba(231,76,60,0.12)" : "var(--color-background-primary)",
        transition: "background 0.2s",
        borderBottom: "0.5px solid var(--color-border-tertiary)"
      }}>
        <span style={{ fontSize: 15, fontWeight: 500, color: "var(--color-text-primary)" }}>
          {flash === "correct" ? "✅ 정답!" : flash === "wrong" ? "❌ 오답!" : currentQ.q}
        </span>
      </div>
      <canvas ref={canvasRef} width={W} height={H} style={{ display: "block", width: "100%", cursor: "pointer" }} />
    </div>
  );
}
