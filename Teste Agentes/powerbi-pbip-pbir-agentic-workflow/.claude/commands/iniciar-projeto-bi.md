---
description: Inicia o pipeline guiado completo de desenvolvimento Power BI (PBIP/PBIR) com gates de aprovação humana.
---

# Iniciar Projeto BI

Você acaba de receber o comando `/iniciar-projeto-bi`.

Invoque imediatamente o agente `orquestrador-bi` em **modo guided pipeline** com a seguinte instrução:

> "Execute o pipeline guiado de desenvolvimento Power BI conforme `.claude/agents/00-orquestrador-bi.md` § Modo Guided Pipeline. Comece pela Fase 0 (Análise do projeto). Use a skill `gate-protocol` em cada gate. Se já houver `TODO.md` com gates `[x]`, ofereça opção de retomar."

Não execute ações fora desse handoff. Após o agente devolver controle (gate aprovado, ajuste pedido ou pausa), aguarde nova instrução do usuário.
