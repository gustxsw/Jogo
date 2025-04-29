const canvas = document.getElementById("gameCanvas");
const ctx = canvas.getContext("2d");

// Função para atualizar o tamanho do canvas dinamicamente
function ajustarTamanhoCanvas() {
  canvas.width = window.innerWidth > 360 ? 360 : window.innerWidth;
  canvas.height = window.innerHeight > 640 ? 640 : window.innerHeight;
}

// Chama a função de ajuste quando a janela é redimensionada
window.addEventListener("resize", ajustarTamanhoCanvas);

// Inicializa o tamanho do canvas
ajustarTamanhoCanvas();

let faseAtual = 1;
let moedasColetadas = 0;
let coracoesColetados = 0;
let fase2Comecou = false;
let faseTerminada = false;
let fase2Timer = 20000;

// Personagem
const personagem = {
  x: canvas.width / 2 - 15,
  y: canvas.height - 60,
  largura: 40,
  altura: 40,
  velocidade: 5,
  sprite: new Image(),
};
personagem.sprite.src = "assets/player1.png";

// Capivara
const capivara = {
  x: 0,
  y: canvas.height - 60,
  largura: 40,
  altura: 40,
  velocidade: 2,
  sprite: new Image(),
  direcao: 1,
};
capivara.sprite.src = "assets/capivara.png";

// Itens
let coracoes = [];
let obstaculos = [];
let moedas = [];
let presentes = [];

// Imagens
const imgCoracao = new Image();
imgCoracao.src = "assets/heart.png";
const imgMoeda = new Image();
imgMoeda.src = "assets/moeda.png";
const imgPresente = new Image();
imgPresente.src = "assets/gift.png";
const imgFaca = new Image();
imgFaca.src = "assets/faca.png";

// Gera corações
function gerarCoracoes() {
  coracoes = [];
  for (let i = 0; i < 5; i++) {
    coracoes.push({
      x: Math.random() * (canvas.width - 30),
      y: Math.random() * -200,
      largura: 30,
      altura: 30,
      velocidade: Math.random() * 2 + 1,
      coletado: false,
    });
  }
}

// Gera itens da fase 2
function gerarItensFase2() {
  setInterval(() => {
    if (faseAtual === 2 && !faseTerminada) {
      obstaculos.push({
        x: Math.random() * (canvas.width - 40),
        y: -50,
        largura: 40,
        altura: 40,
        velocidade: Math.random() * 2 + 2,
        angulo: 0,
      });

      moedas.push({
        x: Math.random() * (canvas.width - 30),
        y: -50,
        largura: 30,
        altura: 30,
        velocidade: Math.random() * 2 + 1,
        coletada: false,
      });

      if (presentes.length < 1) {
        presentes.push({
          x: Math.random() * (canvas.width - 40),
          y: -50,
          largura: 40,
          altura: 40,
          velocidade: Math.random() * 2 + 1,
        });
      }
    }
  }, 1000);
}

// Controles
document.addEventListener("keydown", (e) => {
  if (e.key === "ArrowLeft" && personagem.x > 0) {
    personagem.x -= personagem.velocidade;
  }
  if (
    e.key === "ArrowRight" &&
    personagem.x < canvas.width - personagem.largura
  ) {
    personagem.x += personagem.velocidade;
  }
});

canvas.addEventListener("touchstart", (e) => {
  const toqueX = e.touches[0].clientX;
  if (toqueX < canvas.width / 2) {
    personagem.x -= personagem.velocidade * 3;
  } else {
    personagem.x += personagem.velocidade * 3;
  }
});

// Desenha tudo
function desenharFase() {
  ctx.clearRect(0, 0, canvas.width, canvas.height);

  // UI
  ctx.drawImage(imgMoeda, 10, 10, 30, 30);
  ctx.fillStyle = "yellow";
  ctx.font = "20px Arial";
  ctx.fillText(`${moedasColetadas}`, 50, 32);

  ctx.fillStyle = "red";
  ctx.font = "20px Arial";
  if (faseAtual === 1) {
    ctx.fillText("Fase 1 - Colete os corações!", 50, 50);
  } else if (faseAtual === 2) {
    ctx.fillText("Fase 2 - Desvie das facas!", 50, 50);
  }

  // Itens da fase
  coracoes.forEach((coracao) => {
    if (!coracao.coletado) {
      ctx.drawImage(
        imgCoracao,
        coracao.x,
        coracao.y,
        coracao.largura,
        coracao.altura
      );
    }
  });

  ctx.drawImage(
    capivara.sprite,
    capivara.x,
    capivara.y,
    capivara.largura,
    capivara.altura
  );

  if (faseAtual === 2) {
    obstaculos.forEach((obstaculo) => {
      ctx.save();
      ctx.translate(
        obstaculo.x + obstaculo.largura / 2,
        obstaculo.y + obstaculo.altura / 2
      );
      ctx.rotate((obstaculo.angulo * Math.PI) / 180);
      ctx.drawImage(
        imgFaca,
        -obstaculo.largura / 2,
        -obstaculo.altura / 2,
        obstaculo.largura,
        obstaculo.altura
      );
      ctx.restore();
    });

    presentes.forEach((presente) => {
      ctx.drawImage(
        imgPresente,
        presente.x,
        presente.y,
        presente.largura,
        presente.altura
      );
    });

    moedas.forEach((moeda) => {
      if (!moeda.coletada) {
        ctx.drawImage(imgMoeda, moeda.x, moeda.y, moeda.largura, moeda.altura);
      }
    });
  }

  ctx.drawImage(
    personagem.sprite,
    personagem.x,
    personagem.y,
    personagem.largura,
    personagem.altura
  );

  if (faseTerminada) {
    ctx.fillStyle = "white";
    ctx.font = "25px Arial";
    ctx.fillText("Parabéns! Jogo finalizado!", 50, canvas.height / 2);
  }
}

// Atualiza tudo
function atualizarItens() {
  capivara.x += capivara.velocidade * capivara.direcao;
  if (capivara.x <= 0 || capivara.x + capivara.largura >= canvas.width) {
    capivara.direcao *= -1;
  }

  if (faseAtual === 1) {
    coracoes.forEach((coracao) => {
      if (!coracao.coletado) {
        coracao.y += coracao.velocidade;

        if (
          personagem.x < coracao.x + coracao.largura &&
          personagem.x + personagem.largura > coracao.x &&
          personagem.y < coracao.y + coracao.altura &&
          personagem.y + personagem.altura > coracao.y
        ) {
          coracao.coletado = true;
          coracoesColetados++;
          if (coracoesColetados >= 5) {
            faseAtual = 2;
            gerarItensFase2();
          }
        }
      }
    });

    coracoes = coracoes.filter((c) => !c.coletado);

    while (coracoes.length < 5 && coracoesColetados < 5) {
      coracoes.push({
        x: Math.random() * (canvas.width - 30),
        y: Math.random() * -200,
        largura: 30,
        altura: 30,
        velocidade: Math.random() * 2 + 1,
        coletado: false,
      });
    }
  }

  if (faseAtual === 2 && !faseTerminada) {
    fase2Timer -= 16;

    obstaculos.forEach((o) => {
      o.y += o.velocidade;
      o.angulo += 5;

      if (
        personagem.x < o.x + o.largura &&
        personagem.x + personagem.largura > o.x &&
        personagem.y < o.y + o.altura &&
        personagem.y + personagem.altura > o.y
      ) {
        alert("Game Over! Você foi atingido!");
        document.location.reload();
      }
    });

    moedas.forEach((moeda) => {
      moeda.y += moeda.velocidade;

      if (
        personagem.x < moeda.x + moeda.largura &&
        personagem.x + personagem.largura > moeda.x &&
        personagem.y < moeda.y + moeda.altura &&
        personagem.y + personagem.altura > moeda.y
      ) {
        moeda.coletada = true;
        moedasColetadas++;
      }
    });

    presentes.forEach((presente) => {
      presente.y += presente.velocidade;
    });

    if (fase2Timer <= 0) {
      faseTerminada = true;
    }
  }
}

// Loop
function loop() {
  desenharFase();
  atualizarItens();
  requestAnimationFrame(loop);
}

gerarCoracoes();
loop();
