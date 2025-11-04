# Robo_Sensores
Trabalho de Robótica sensorial, utilizando sensores de proximidade 


# Documentação do Projeto: NAO - Desvio de Obstáculos Reativo

  - Visão Geral do Projeto
 
 Este projeto implementa um controle de desvio de obstáculos reativo simples para o robô humanóide NAO dentro do simulador CoppeliaSim (V-REP). O objetivo principal é demonstrar uma arquitetura de controle reativa de baixo nível, permitindo que o NAO navegue por um ambiente com pilares (obstáculos) usando dados de três sensores de proximidade adicionais, sem depender de um mapa global ou planejamento complexo. O código-fonte (em Lua) é injetado no objeto raiz do NAO e utiliza as funções API do CoppeliaSim para leitura de sensores e movimentação do robô. 
 
  - Pré-requisitos e Setup:
 
 Para replicar este projeto, você precisará dos seguintes softwares:
 
   CoppeliaSim EDU (ou versão Pro)
     Nota: O projeto foi desenvolvido na versão 4.x do simulador, que utiliza scripts em Lua para controle.
     
   Modelo do Robô NAO (incluído nas bibliotecas padrão do CoppeliaSim).
   
  - Estrutura da Cena:
   
  A cena deve conter:
     O Robô NAO (objeto raiz /NAO).
     Três Sensores de Proximidade (de preferência do tipo cônico, como visto na imagem) montados na frente do robô:
     
       proximitySensor1 (Direita)
       proximitySensor2 (Esquerda)
       proximitySensor3 (Central)
    Obstáculos (no caso, pilares de geometria simples) dispostos no ambiente.
 
 - Lógica de Decisão(sysCall_sensing):
 
  O robô prioriza o desvio para a direita quando detecta obstáculos à frente ou à esquerda, e desvia para a esquerda apenas quando o obstáculo está unicamente à direita.O desvio segue a seguinte lógica de prioridade (decisão tomada no primeiro if verdadeiro):Obstáculo Central (res3 > 0): Prioridade máxima. Gira para a direita (girar(-1)).Obstáculo Esquerdo (res2 > 0): Segunda prioridade. Gira para a direita (girar(-1)).Obstáculo Direito (res1 > 0): Terceira prioridade. Gira para a esquerda (girar(1)).Caminho Livre (Nenhuma Detecção): Move-se para frente (moverParaFrente()).

- Código:
      
      -- Sensores
      local proximitySensor1 -- direita
      local proximitySensor2 -- esquerda
      local proximitySensor3 -- central
      
      -- Parâmetros de movimento
      local VELOCIDADE_AVANCO = 0.01   -- deslocamento para frente por passo
      local VELOCIDADE_GIRO   = 0.05   -- giro por passo (rad)
      local nao
      
      -- Inicialização
      function sysCall_init()
          nao = sim.getObject('/NAO')
      
          -- Mapeamento dos Sensores
          proximitySensor1 = sim.getObject('/NAO/proximitySensor1') -- direita
          proximitySensor2 = sim.getObject('/NAO/proximitySensor2') -- esquerda
          proximitySensor3 = sim.getObject('/NAO/proximitySensor3') -- central
      
          sim.addLog(sim.verbosity_scriptinfos, "Controle de desvio seguro inicializado.")
      end
      
      -- Função para mover o robô para frente
      function moverParaFrente()
          local pos = sim.getObjectPosition(nao, -1)
          local orient = sim.getObjectOrientation(nao, -1)
          -- Calcula novo X e Y baseado na orientação (yaw) do robô
          pos[1] = pos[1] + VELOCIDADE_AVANCO * math.cos(orient[3])
          pos[2] = pos[2] + VELOCIDADE_AVANCO * math.sin(orient[3])
          sim.setObjectPosition(nao, -1, pos)
      end
      
      -- Função para girar o robô
      function girar(direcao)
          -- direcao: 1 = esquerda (aumenta o yaw), -1 = direita (diminui o yaw)
          local orient = sim.getObjectOrientation(nao, -1)
          orient[3] = orient[3] + direcao * VELOCIDADE_GIRO
          sim.setObjectOrientation(nao, -1, orient)
      end
      
      -- Loop de controle (Executado a cada passo de simulação)
      function sysCall_sensing()
          local res1, dist1 = sim.readProximitySensor(proximitySensor1)
          local res2, dist2 = sim.readProximitySensor(proximitySensor2)
          local res3, dist3 = sim.readProximitySensor(proximitySensor3)
      
          -- Decisão de desvio (Lógica Reativa com Prioridade)
          if res3 > 0 then
              -- Central detecta: Gira para direita
              sim.addLog(sim.verbosity_scriptinfos, "Obstáculo central -> desviando para direita")
              girar(-1)
      
          elseif res2 > 0 then
              -- Esquerdo detecta: Gira para direita
              sim.addLog(sim.verbosity_scriptinfos, "Obstáculo à esquerda -> desviando para direita")
              girar(-1)
      
          elseif res1 > 0 then
              -- Direito detecta: Gira para esquerda
              sim.addLog(sim.verbosity_scriptinfos, "Obstáculo à direita -> desviando para esquerda")
              girar(1)
      
          else
              -- Caminho livre: Anda para frente
              moverParaFrente()
          end
      end
      
      -- Finalização
      function sysCall_cleanup()
          sim.addLog(sim.verbosity_scriptinfos, "Sistema de desvio seguro finalizado.")
      end
