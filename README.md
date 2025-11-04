# Robo_Sensores
Trabalho de Robótica sensorial, utilizando sensores de proximidade 


# Documentação do Projeto: NAO - Desvio de Obstáculos Reativo

   Visão Geral do Projeto
 
 Este projeto implementa um controle de desvio de obstáculos reativo simples para o robô humanóide NAO dentro do simulador CoppeliaSim (V-REP). O objetivo principal é demonstrar uma arquitetura de controle reativa de baixo nível, permitindo que o NAO navegue por um ambiente com pilares (obstáculos) usando dados de três sensores de proximidade adicionais, sem depender de um mapa global ou planejamento complexo. O código-fonte (em Lua) é injetado no objeto raiz do NAO e utiliza as funções API do CoppeliaSim para leitura de sensores e movimentação do robô. 
 
   Pré-requisitos e Setup:
 
 Para replicar este projeto, você precisará dos seguintes softwares:
 
   CoppeliaSim EDU (ou versão Pro)
     Nota: O projeto foi desenvolvido na versão 4.x do simulador, que utiliza scripts em Lua para controle.
     
   Modelo do Robô NAO (incluído nas bibliotecas padrão do CoppeliaSim).
   
   Estrutura da Cena:
   
   A cena deve conter:
     O Robô NAO (objeto raiz /NAO).
     Três Sensores de Proximidade (de preferência do tipo cônico, como visto na imagem) montados na frente do robô:
     
       proximitySensor1 (Direita)
       proximitySensor2 (Esquerda)
       proximitySensor3 (Central)
    Obstáculos (no caso, pilares de geometria simples) dispostos no ambiente.
 
  Lógica de Decisão(sysCall_sensing):
 
  O robô prioriza o desvio para a direita quando detecta obstáculos à frente ou à esquerda, e desvia para a esquerda apenas quando o obstáculo está unicamente à direita.O desvio segue a seguinte lógica de prioridade (decisão tomada no primeiro if verdadeiro):Obstáculo Central (res3 > 0): Prioridade máxima. Gira para a direita (girar(-1)).Obstáculo Esquerdo (res2 > 0): Segunda prioridade. Gira para a direita (girar(-1)).Obstáculo Direito (res1 > 0): Terceira prioridade. Gira para a esquerda (girar(1)).Caminho Livre (Nenhuma Detecção): Move-se para frente (moverParaFrente()).
