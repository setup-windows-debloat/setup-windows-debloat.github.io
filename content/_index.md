---
title: "🌌 Windows Debloat "
date: 2026-07-08
draft: false
---

# 🌌 Windows Debloat

Windows Debloat, refere-se ao processo de remoção de aplicativos e serviços pré-instalados no Windows 10 que não são utilizados, visando otimizar o desempenho do sistema e aumentar a privacidade.

## Definição

O termo "debloat" significa "desinchar" e, no contexto do Windows, refere-se à remoção de bloatware, que são aplicativos e serviços que vêm pré-instalados no sistema operacional, mas que muitas vezes não são necessários para o usuário. Esses aplicativos podem ocupar espaço em disco, consumir recursos do sistema e coletar dados sobre os hábitos de uso do usuário. Ao realizar o debloat, você pode liberar memória RAM e espaço em disco, além de melhorar a privacidade ao reduzir a quantidade de informações coletadas por esses aplicativos


## Benefícios
- Melhoria no Desempenho: Remover aplicativos desnecessários pode ajudar a reduzir a carga sobre o sistema, resultando em um desempenho mais rápido e eficiente.
- Aumento da Privacidade: Ao eliminar aplicativos que coletam dados, você pode proteger melhor suas informações pessoais.
- Liberação de Espaço: O debloat ajuda a liberar espaço em disco, permitindo que você armazene arquivos e aplicativos mais importantes


## Considerações Finais
Antes de realizar o debloat, é recomendável criar um ponto de restauração no sistema, para que você possa reverter as alterações caso algo não funcione como esperado. O debloat pode ser uma maneira eficaz de personalizar e otimizar sua experiência no Windows 10, tornando o sistema mais leve e adaptado às suas necessidade

---

### 📋 O que o Script Faz? (Resumo das Etapas)

* **Expurga e Bloqueia a Telemetria:** Interrompe permanentemente os serviços de coleta de dados da Microsoft (como `DiagTrack` e `WerSvc`), blindando a privacidade do sistema.
* **Aniquila o Consumo Fantasma de RAM:** Desativa subsistemas pesados como o `SysMain` (Superfetch) e o indexador `Windows Search`, eliminando picos de leitura e escrita em SSDs/NVMes.
* **Destrói Recursos Cloud Ocultos:** Sepulta a Cortana, desativa as pesquisas do Bing integradas ao menu iniciar e remove o pacote de Widgets (`WebExperience`).
* **Maximiza os Efeitos Visuais:** Configura o sistema para o modo de "Melhor Desempenho", eliminando transições, sombras e transparências desnecessárias.
* **Executa a Higienização da Memória Física:** Injeta código C# nativo diretamente no Kernel NT para esvaziar o *Working Set* de todos os processos zumbis em execução.
* **Silencia Aplicativos em Segundo Plano:** Aplica restrições estritas no registro do Windows para bloquear o funcionamento em background de mais de 30 pacotes (incluindo Copilot, Xbox, Spotify e Teams).
* **Blinda o Navegador Edge:** Cria políticas locais rígidas (`HKLM`) para impedir que o Microsoft Edge pré-carregue ou execute extensões secretamente.

---

### 📊 Tabela de Impacto: Antes vs. Depois

| Componente do Sistema | Estado Original (Invasivo/Pesado) | Estado Modificado (Otimizado/Estável) | Benefício Direto Obtido |
| :--- | :--- | :--- | :--- |
| **Coleta de Diagnósticos** | Telemetria ativa enviando dados (`DiagTrack`). | **Desativada permanentemente**. | Privacidade blindada e economia de banda. |
| **Gerenciamento de Cache** | `SysMain` e `WSearch` consumindo RAM e Disco. | **Serviços abortados e congelados**. | Fim dos engasgos (*stuttering*) em jogos e código. |
| **Menu Iniciar & Barra** | Widgets de notícias e buscas do Bing ativos. | **Interface limpa e local**. | Abertura instantânea do menu de busca. |
| **Efeitos de Interface** | Animações e transparências pesadas. | **Estilo clássico e otimizado**. | Redução imediata na latência de renderização de DWM. |
| **Microsoft Edge** | Processos zumbis pré-carregados na inicialização. | **Bloqueio total em segundo plano**. | Liberação instantânea de megabytes preciosos de RAM. |
| **Recurso Windows Recall** | Rastreamento contínuo de tela ativo. | **Recurso removido via DISM**. | Segurança de dados elevada ao nível máximo. |

---

# 📥 Área de Downloads

Clique no botão abaixo para baixar o script operacional de automação diretamente para a sua máquina:

[📥 Baixar windows-debloat.ps1](/files/windows-debloat.ps1){.btn-download download="windows-debloat.ps1"}

---
*Página gerada estaticamente com foco em alta performance e legibilidade noturna.*
