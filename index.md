<!DOCTYPE html>
<html lang="pt-BR">

<head>

  <meta charset="utf-8">

  <meta
    name="viewport"
    content="width=device-width,
             initial-scale=1.0,
             maximum-scale=1.0,
             user-scalable=no,
             viewport-fit=cover">

  <title>Lago VR - Mobile Cardboard</title>


  <!-- =====================================================
       A-FRAME
  ====================================================== -->

  <script src="https://aframe.io/releases/1.4.0/aframe.min.js"></script>


  <style>

    html,
    body {

      margin: 0;
      padding: 0;

      width: 100%;
      height: 100%;

      overflow: hidden;

      background: #000;

      touch-action: none;

    }


    /*
     * Botão inicial
     */

    #iniciar {

      position: fixed;

      left: 50%;
      top: 50%;

      transform: translate(-50%, -50%);

      z-index: 9999;

      padding: 18px 28px;

      border: none;

      border-radius: 14px;

      background: #2196F3;

      color: white;

      font-size: 20px;

      font-weight: bold;

      box-shadow:
        0 5px 20px rgba(0,0,0,0.4);

      cursor: pointer;

      -webkit-tap-highlight-color: transparent;

    }


    #iniciar:active {

      transform:
        translate(-50%, -50%)
        scale(0.96);

    }


    /*
     * Pequena mensagem de orientação
     */

    #mensagem {

      position: fixed;

      left: 50%;

      bottom: 25px;

      transform: translateX(-50%);

      z-index: 9998;

      width: 85%;

      max-width: 400px;

      padding: 12px 16px;

      box-sizing: border-box;

      border-radius: 10px;

      background: rgba(0,0,0,0.65);

      color: white;

      text-align: center;

      font-size: 14px;

      line-height: 1.4;

      display: none;

    }

  </style>


  <script>


    // =========================================================
    // CILINDRO
    //
    // Olhar para o cilindro durante 1 segundo.
    // Enquanto continuar olhando, ele muda de cor
    // a cada 1 segundo.
    // =========================================================

    AFRAME.registerComponent("muda-cor-olhar", {

      schema: {

        tempo: {
          type: "number",
          default: 1000
        }

      },


      init: function () {

        const el = this.el;


        const cores = [

          "#FF0000", // Vermelho
          "#00FF00", // Verde
          "#0000FF", // Azul
          "#FFFF00", // Amarelo
          "#00FFFF", // Ciano
          "#FF00FF", // Magenta
          "#FF9800", // Laranja
          "#800080"  // Roxo

        ];


        let timer = null;

        let indice = 0;

        let olhando = false;


        // -----------------------------------------------------
        // Começou a olhar
        // -----------------------------------------------------

        el.addEventListener("mouseenter", function () {

          olhando = true;


          if (timer) {
            return;
          }


          function mudarCor() {

            if (!olhando) {

              timer = null;

              return;

            }


            indice++;


            if (indice >= cores.length) {

              indice = 0;

            }


            el.setAttribute(
              "material",
              "color",
              cores[indice]
            );


            timer = setTimeout(
              mudarCor,
              1000
            );

          }


          timer = setTimeout(
            mudarCor,
            1000
          );

        });


        // -----------------------------------------------------
        // Parou de olhar
        // -----------------------------------------------------

        el.addEventListener("mouseleave", function () {

          olhando = false;


          if (timer) {

            clearTimeout(timer);

            timer = null;

          }

        });

      }

    });



    // =========================================================
    // PEDRA
    //
    // Olhar para a pedra durante 1 segundo.
    // A pedra pula.
    // =========================================================

    AFRAME.registerComponent("pedra-olhar", {

      init: function () {

        const el = this.el;

        let timer = null;


        el.addEventListener("mouseenter", function () {

          if (timer) {
            return;
          }


          timer = setTimeout(function () {


            el.setAttribute("animation__pulo", {

              property: "position",

              from: "-3 0.6 -4",

              to: "-3 1.8 -4",

              dur: 400,

              dir: "alternate",

              loop: 2,

              easing: "easeOutQuad"

            });


            timer = null;


          }, 1000);

        });


        el.addEventListener("mouseleave", function () {

          if (timer) {

            clearTimeout(timer);

            timer = null;

          }

        });

      }

    });



    // =========================================================
    // INICIALIZAÇÃO MOBILE
    //
    // Solicita acesso aos sensores quando o navegador
    // exigir permissão.
    // =========================================================

    async function iniciarExperiencia() {

      const botao =
        document.querySelector("#iniciar");


      const mensagem =
        document.querySelector("#mensagem");


      botao.style.display = "none";


      mensagem.style.display = "block";


      mensagem.innerText =
        "Preparando a experiência VR...";


      try {


        // -----------------------------------------------------
        // iPhone / iPad
        //
        // Alguns navegadores exigem uma permissão explícita
        // para utilizar os sensores de movimento.
        // -----------------------------------------------------

        if (

          typeof DeviceOrientationEvent !==
          "undefined"

          &&

          typeof DeviceOrientationEvent.requestPermission ===
          "function"

        ) {


          const permissao =
            await DeviceOrientationEvent.requestPermission();


          if (permissao !== "granted") {

            mensagem.innerText =
              "Acesso aos sensores foi negado. Permita o acesso aos sensores e tente novamente.";

            botao.style.display = "block";

            return;

          }

        }


        mensagem.innerText =
          "Sensores ativados. Prepare o óculos VR.";


        // -----------------------------------------------------
        // Aguarda a cena carregar
        // -----------------------------------------------------

        const cena =
          document.querySelector("a-scene");


        if (!cena.hasLoaded) {

          await new Promise(function(resolve) {

            cena.addEventListener(
              "loaded",
              resolve,
              { once: true }
            );

          });

        }


        // -----------------------------------------------------
        // Tenta entrar no modo VR
        //
        // O A-Frame utiliza WebXR quando disponível.
        // -----------------------------------------------------

        if (
          typeof cena.enterVR === "function"
        ) {

          try {

            await cena.enterVR();

          } catch (erro) {

            console.log(
              "Não foi possível entrar automaticamente no modo VR:",
              erro
            );

          }

        }


        mensagem.style.display = "none";


      } catch (erro) {

        console.log(
          "Erro ao iniciar experiência:",
          erro
        );


        mensagem.innerText =
          "Não foi possível acessar os sensores do celular.";


        botao.style.display = "block";

      }

    }



    // =========================================================
    // QUANDO A PÁGINA CARREGAR
    // =========================================================

    document.addEventListener(
      "DOMContentLoaded",
      function () {


        const botao =
          document.querySelector("#iniciar");


        botao.addEventListener(
          "click",
          iniciarExperiencia
        );


      }
    );


  </script>

</head>



<body>


<!-- =======================================================
     BOTÃO INICIAL
======================================================== -->

<button id="iniciar">

  🚀 INICIAR VR

</button>


<!-- =======================================================
     MENSAGEM MOBILE
======================================================== -->

<div id="mensagem"></div>



<!-- =======================================================
     CENA VR
======================================================== -->

<a-scene

  background="color: #87CEEB"

  shadow="type: basic"

  renderer="
    antialias: true;
    colorManagement: true
  "

  vr-mode-ui="
    enabled: true
  "

  device-orientation-permission-ui="
    enabled: true
  ">


  <!-- =====================================================
       CÉU
  ====================================================== -->

  <a-sky
    color="#87CEEB">
  </a-sky>



  <!-- =====================================================
       TERRENO DO PARQUE
  ====================================================== -->

  <a-plane

    position="0 0 0"

    rotation="-90 0 0"

    width="24"

    height="24"

    color="#4CAF50"

    shadow="receive: true">

  </a-plane>



  <!-- =====================================================
       CAMINHO CENTRAL
  ====================================================== -->

  <a-plane

    position="0 0.025 2"

    rotation="-90 0 0"

    width="3"

    height="20"

    color="#C9A66B">

  </a-plane>



  <!-- =====================================================
       LAGO
  ====================================================== -->

  <a-circle

    position="0 0.05 -6"

    rotation="-90 0 0"

    radius="5"

    color="#0077BE"

    opacity="0.9">

  </a-circle>



  <!-- =====================================================
       PEDRAS DO LAGO
  ====================================================== -->

  <a-sphere

    position="-4.3 0.18 -5"

    radius="0.3"

    color="#777777">

  </a-sphere>


  <a-sphere

    position="4.2 0.18 -5"

    radius="0.3"

    color="#777777">

  </a-sphere>


  <a-sphere

    position="-2.5 0.15 -9.5"

    radius="0.25"

    color="#888888">

  </a-sphere>


  <a-sphere

    position="2.5 0.15 -9.5"

    radius="0.25"

    color="#888888">

  </a-sphere>



  <!-- =====================================================
       PEDRA INTERATIVA
  ====================================================== -->

  <a-dodecahedron

    id="pedra"

    pedra-olhar

    class="raycastable"

    position="-3 0.6 -4"

    radius="0.6"

    color="#888888"

    shadow="cast: true">

  </a-dodecahedron>



  <!-- =====================================================
       CILINDRO INTERATIVO
  ====================================================== -->

  <a-cylinder

    id="cilindro-magico"

    muda-cor-olhar

    class="raycastable"

    position="0 1 -4"

    radius="0.5"

    height="1.8"

    color="#FF0000"

    shadow="cast: true">

  </a-cylinder>



  <!-- =====================================================
       PATO
       
       Agora ele NÃO possui nenhuma interação por olhar.
       
       Ele simplesmente fica pulando continuamente.
  ====================================================== -->

  <a-entity

    id="pato-group"

    position="3.5 0.4 -4"

    rotation="0 -45 0"

    shadow="cast: true"

    animation__pulo="

      property: position;

      from: 3.5 0.4 -4;

      to: 3.5 1.2 -4;

      dur: 400;

      dir: alternate;

      loop: true;

      easing: easeInOutQuad

    ">


    <!-- Corpo -->

    <a-sphere

      radius="0.3"

      color="#FFEB3B"

      scale="1 0.8 1.2">

    </a-sphere>


    <!-- Cabeça -->

    <a-sphere

      position="0 0.3 0.2"

      radius="0.18"

      color="#FFEB3B">

    </a-sphere>


    <!-- Bico -->

    <a-cone

      position="0 0.28 0.4"

      rotation="90 0 0"

      radius-bottom="0.08"

      height="0.15"

      color="#FF9800">

    </a-cone>


    <!-- Olho direito -->

    <a-sphere

      position="0.1 0.35 0.3"

      radius="0.03"

      color="#000000">

    </a-sphere>


    <!-- Olho esquerdo -->

    <a-sphere

      position="-0.1 0.35 0.3"

      radius="0.03"

      color="#000000">

    </a-sphere>


  </a-entity>



  <!-- =====================================================
       ÁRVORES GRANDES
  ====================================================== -->


  <!-- Árvore 1 -->

  <a-entity position="-9 0 -9">

    <a-cylinder

      position="0 2 0"

      radius="0.45"

      height="4"

      color="#5D4037"

      shadow="cast: true">

    </a-cylinder>


    <a-sphere

      position="0 4.3 0"

      radius="2"

      color="#1B5E20"

      shadow="cast: true">

    </a-sphere>


    <a-sphere

      position="-1.2 4.1 0"

      radius="1.3"

      color="#2E7D32">

    </a-sphere>


    <a-sphere

      position="1.2 4.1 0"

      radius="1.3"

      color="#388E3C">

    </a-sphere>

  </a-entity>



  <!-- Árvore 2 -->

  <a-entity position="9 0 -9">

    <a-cylinder

      position="0 2 0"

      radius="0.45"

      height="4"

      color="#5D4037"

      shadow="cast: true">

    </a-cylinder>


    <a-sphere

      position="0 4.3 0"

      radius="2"

      color="#1B5E20"

      shadow="cast: true">

    </a-sphere>


    <a-sphere

      position="-1.2 4.1 0"

      radius="1.3"

      color="#2E7D32">

    </a-sphere>


    <a-sphere

      position="1.2 4.1 0"

      radius="1.3"

      color="#388E3C">

    </a-sphere>

  </a-entity>



  <!-- Árvore 3 -->

  <a-entity position="-9 0 -2">

    <a-cylinder

      position="0 1.7 0"

      radius="0.35"

      height="3.4"

      color="#795548"

      shadow="cast: true">

    </a-cylinder>


    <a-sphere

      position="0 3.8 0"

      radius="1.6"

      color="#2E7D32"

      shadow="cast: true">

    </a-sphere>


    <a-sphere

      position="-1 3.6 0"

      radius="1"

      color="#388E3C">

    </a-sphere>


    <a-sphere

      position="1 3.6 0"

      radius="1"

      color="#388E3C">

    </a-sphere>

  </a-entity>



  <!-- Árvore 4 -->

  <a-entity position="9 0 -2">

    <a-cylinder

      position="0 1.7 0"

      radius="0.35"

      height="3.4"

      color="#795548"

      shadow="cast: true">

    </a-cylinder>


    <a-sphere

      position="0 3.8 0"

      radius="1.6"

      color="#2E7D32"

      shadow="cast: true">

    </a-sphere>


    <a-sphere

      position="-1 3.6 0"

      radius="1"

      color="#388E3C">

    </a-sphere>


    <a-sphere

      position="1 3.6 0"

      radius="1"

      color="#388E3C">

    </a-sphere>

  </a-entity>



  <!-- Árvore 5 -->

  <a-entity position="-8 0 7">

    <a-cylinder

      position="0 1.8 0"

      radius="0.4"

      height="3.6"

      color="#6D4C41"

      shadow="cast: true">

    </a-cylinder>


    <a-sphere

      position="0 4 0"

      radius="1.7"

      color="#1B5E20"

      shadow="cast: true">

    </a-sphere>


    <a-sphere

      position="-1 3.8 0"

      radius="1.1"

      color="#2E7D32">

    </a-sphere>


    <a-sphere

      position="1 3.8 0"

      radius="1.1"

      color="#388E3C">

    </a-sphere>

  </a-entity>



  <!-- Árvore 6 -->

  <a-entity position="8 0 7">

    <a-cylinder

      position="0 1.8 0"

      radius="0.4"

      height="3.6"

      color="#6D4C41"

      shadow="cast: true">

    </a-cylinder>


    <a-sphere

      position="0 4 0"

      radius="1.7"

      color="#1B5E20"

      shadow="cast: true">

    </a-sphere>


    <a-sphere

      position="-1 3.8 0"

      radius="1.1"

      color="#2E7D32">

    </a-sphere>


    <a-sphere

      position="1 3.8 0"

      radius="1.1"

      color="#388E3C">

    </a-sphere>

  </a-entity>



  <!-- Árvore 7 -->

  <a-entity position="-5 0 5">

    <a-cylinder

      position="0 1.5 0"

      radius="0.3"

      height="3"

      color="#795548">

    </a-cylinder>


    <a-sphere

      position="0 3.4 0"

      radius="1.4"

      color="#388E3C">

    </a-sphere>

  </a-entity>



  <!-- Árvore 8 -->

  <a-entity position="5 0 5">

    <a-cylinder

      position="0 1.5 0"

      radius="0.3"

      height="3"

      color="#795548">

    </a-cylinder>


    <a-sphere

      position="0 3.4 0"

      radius="1.4"

      color="#388E3C">

    </a-sphere>

  </a-entity>



  <!-- =====================================================
       BANCO
  ====================================================== -->

  <a-entity

    position="5 0 1"

    rotation="0 0 0">


    <!-- Assento -->

    <a-box

      position="0 0.85 0"

      width="2.4"

      height="0.22"

      depth="0.65"

      color="#8D6E63"

      shadow="cast: true">

    </a-box>


    <!-- Encosto -->

    <a-box

      position="0 1.25 0.25"

      width="2.4"

      height="0.8"

      depth="0.18"

      color="#6D4C41"

      shadow="cast: true">

    </a-box>


    <!-- Pé esquerdo -->

    <a-box

      position="-0.85 0.4 0"

      width="0.18"

      height="0.8"

      depth="0.45"

      color="#263238"

      shadow="cast: true">

    </a-box>


    <!-- Pé direito -->

    <a-box

      position="0.85 0.4 0"

      width="0.18"

      height="0.8"

      depth="0.45"

      color="#263238"

      shadow="cast: true">

    </a-box>


    <!-- Apoio inferior esquerdo -->

    <a-box

      position="-0.85 0.25 0"

      width="0.25"

      height="0.12"

      depth="0.7"

      color="#263238">

    </a-box>


    <!-- Apoio inferior direito -->

    <a-box

      position="0.85 0.25 0"

      width="0.25"

      height="0.12"

      depth="0.7"

      color="#263238">

    </a-box>


  </a-entity>



  <!-- =====================================================
       FLORES
  ====================================================== -->


  <!-- Flor 1 -->

  <a-entity position="-6 0 2">

    <a-cylinder

      position="0 0.3 0"

      radius="0.05"

      height="0.6"

      color="#388E3C">

    </a-cylinder>


    <a-sphere

      position="0 0.65 0"

      radius="0.22"

      color="#E91E63">

    </a-sphere>

  </a-entity>



  <!-- Flor 2 -->

  <a-entity position="-4 0 4">

    <a-cylinder

      position="0 0.3 0"

      radius="0.05"

      height="0.6"

      color="#388E3C">

    </a-cylinder>


    <a-sphere

      position="0 0.65 0"

      radius="0.22"

      color="#FFEB3B">

    </a-sphere>

  </a-entity>



  <!-- Flor 3 -->

  <a-entity position="6 0 3">

    <a-cylinder

      position="0 0.3 0"

      radius="0.05"

      height="0.6"

      color="#388E3C">

    </a-cylinder>


    <a-sphere

      position="0 0.65 0"

      radius="0.22"

      color="#9C27B0">

    </a-sphere>

  </a-entity>



  <!-- Flor 4 -->

  <a-entity position="4 0 6">

    <a-cylinder

      position="0 0.3 0"

      radius="0.05"

      height="0.6"

      color="#388E3C">

    </a-cylinder>


    <a-sphere

      position="0 0.65 0"

      radius="0.22"

      color="#FF5722">

    </a-sphere>

  </a-entity>



  <!-- Flor 5 -->

  <a-entity position="-7 0 -5">

    <a-cylinder

      position="0 0.3 0"

      radius="0.05"

      height="0.6"

      color="#388E3C">

    </a-cylinder>


    <a-sphere

      position="0 0.65 0"

      radius="0.22"

      color="#FFFFFF">

    </a-sphere>

  </a-entity>



  <!-- Flor 6 -->

  <a-entity position="7 0 -6">

    <a-cylinder

      position="0 0.3 0"

      radius="0.05"

      height="0.6"

      color="#388E3C">

    </a-cylinder>


    <a-sphere

      position="0 0.65 0"

      radius="0.22"

      color="#F44336">

    </a-sphere>

  </a-entity>



  <!-- Flor 7 -->

  <a-entity position="-8 0 4">

    <a-cylinder

      position="0 0.3 0"

      radius="0.05"

      height="0.6"

      color="#388E3C">

    </a-cylinder>


    <a-sphere

      position="0 0.65 0"

      radius="0.22"

      color="#FFEB3B">

    </a-sphere>

  </a-entity>



  <!-- Flor 8 -->

  <a-entity position="8 0 4">

    <a-cylinder

      position="0 0.3 0"

      radius="0.05"

      height="0.6"

      color="#388E3C">

    </a-cylinder>


    <a-sphere

      position="0 0.65 0"

      radius="0.22"

      color="#E91E63">

    </a-sphere>

  </a-entity>



  <!-- Flor 9 -->

  <a-entity position="6 0 1">

    <a-cylinder

      position="0 0.3 0"

      radius="0.05"

      height="0.6"

      color="#388E3C">

    </a-cylinder>


    <a-sphere

      position="0 0.65 0"

      radius="0.22"

      color="#FF9800">

    </a-sphere>

  </a-entity>



  <!-- =====================================================
       LUMINÁRIAS
  ====================================================== -->

  <a-entity position="-6 0 -2">

    <a-cylinder

      position="0 1.5 0"

      radius="0.08"

      height="3"

      color="#263238">

    </a-cylinder>


    <a-sphere

      position="0 3.1 0"

      radius="0.2"

      color="#FFF59D">

    </a-sphere>

  </a-entity>



  <a-entity position="6 0 -2">

    <a-cylinder

      position="0 1.5 0"

      radius="0.08"

      height="3"

      color="#263238">

    </a-cylinder>


    <a-sphere

      position="0 3.1 0"

      radius="0.2"

      color="#FFF59D">

    </a-sphere>

  </a-entity>



  <!-- =====================================================
       VEGETAÇÃO EXTRA
  ====================================================== -->

  <a-sphere

    position="-9 0.5 1"

    radius="0.6"

    color="#43A047">

  </a-sphere>


  <a-sphere

    position="9 0.5 1"

    radius="0.6"

    color="#43A047">

  </a-sphere>


  <a-sphere

    position="-7 0.45 8"

    radius="0.5"

    color="#66BB6A">

  </a-sphere>


  <a-sphere

    position="7 0.45 8"

    radius="0.5"

    color="#66BB6A">

  </a-sphere>


  <a-sphere

    position="-9 0.45 -6"

    radius="0.5"

    color="#66BB6A">

  </a-sphere>


  <a-sphere

    position="9 0.45 -6"

    radius="0.5"

    color="#66BB6A">

  </a-sphere>



  <!-- =====================================================
       JOGADOR / CÂMERA
       
       IMPORTANTE:
       
       Não existe mais "rig" com movimento.
       
       A câmera permanece parada no centro.
       
       A orientação da câmera é controlada pelo celular.
  ====================================================== -->

  <a-camera

    id="camera"

    position="0 1.6 0"

    look-controls="
      enabled: true;
      magicWindowTrackingEnabled: true;
      touchEnabled: true
    ">


    <!-- =================================================
         CURSOR CENTRAL
         
         O usuário olha para o objeto.
         
         Depois de 1 segundo:
         
         PEDRA → pula
         CILINDRO → muda de cor
         ================================================= -->

    <a-cursor

      id="cursor"

      color="#FFFFFF"

      fuse="true"

      fuse-timeout="1000"

      raycaster="
        objects: .raycastable
      "

      animation__mouseenter="

        property: scale;

        startEvents: mouseenter;

        from: 1 1 1;

        to: 1.5 1.5 1.5;

        dur: 150

      "

      animation__mouseleave="

        property: scale;

        startEvents: mouseleave;

        from: 1.5 1.5 1.5;

        to: 1 1 1;

        dur: 150

      ">

    </a-cursor>


  </a-camera>


</a-scene>


</body>

</html>
