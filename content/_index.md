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

## 🎛️ Componentes e Diretórios Manipulados pelo Script

Abaixo está o mapeamento detalhado de cada componente vital do sistema operacional que sofre intervenção direta, segmentado por sua respectiva área de atuação no Kernel e Registro do Windows.

### 🛡️ 1. Serviços do Sistema (Services.msc)
O script interrompe o ciclo de vida e altera o tipo de inicialização para **Desativado** (`Disabled`) dos seguintes serviços:
* **`DiagTrack` & `dmwappushservice`:** Componentes primários de telemetria e rastreamento de experiência do usuário.
* **`SysMain`:** Antigo *Superfetch*, responsável pelo cache agressivo de memória que gera picos de uso.
* **`WSearch`:** O indexador nativo do *Windows Search*, mitigando o desgaste e leitura/escrita contínua em SSDs.
* **`WinDefend`:** Mecanismo de proteção em tempo real do *Windows Defender* (desativado para ganho bruto de performance).
* **`DoSvc` & `BITS`:** Serviços de *Otimização de Entrega* e transferência em segundo plano usados pelo Windows Update.
* **`WerSvc`:** Relatório de erros do Windows (*Windows Error Reporting*).
* **Serviços Xbox (`XblAuthManager`, `XblGameSave`, `XboxNetApiSvc`):** Subsistema de jogos em background.

### 🗂️ 2. Chaves de Registro Modificadas (Regedit)
O script realiza injeções e modificações estritas de valores binários e DWORD nas seguintes árvores estruturais:

* **Efeitos Visuais e Desempenho:**
  * `HKCU:\Software\Microsoft\Windows\CurrentVersion\Explorer\VisualEffects` 
  * `HKCU:\Control Panel\Desktop` *(Ajusta a máscara de preferências do usuário e desativa a animação de janelas `MinAnimate`)*.
* **Privacidade e Bloqueio de Telemetria:**
  * `HKLM:\SOFTWARE\Policies\Microsoft\Windows\DataCollection` *(Cria a diretiva de grupo `AllowTelemetry = 0`)*.
  * `HKCU:\Software\Microsoft\Clipboard` *(Sepulta o histórico de transferência em nuvem e sincronização entre dispositivos)*.
  * `HKCU:\Software\Microsoft\Windows\CurrentVersion\CrossDeviceResume\Configuration` *(Aborta o recurso de continuar atividades em outros aparelhos)*.
* **Interface e Menu Iniciar:**
  * `HKCU:\Software\Policies\Microsoft\Windows\Explorer` *(Garante a remoção de sugestões e anúncios na caixa de pesquisa)*.
  * `HKCU:\Software\Microsoft\Windows\CurrentVersion\Feeds` *(Desativa completamente os Widgets de notícias e tempo da barra de tarefas)*.

### 📦 3. Aplicativos Modernos e Recursos Universais (AppX / UWP)
O script executa rotinas de purga e remoção forçada de pacotes integrados utilizando o gerenciador do Windows:
* **`Microsoft.YourPhone`:** Desinstalação completa do aplicativo *Vínculo com o Celular*.
* **`Microsoft.WindowsStore`:** Bloqueio severo de execução em segundo plano para a loja nativa de aplicativos.
* **`Microsoft.Copilot_8wekyb3d8bbwe`:** Desativação do ciclo de vida em background da inteligência artificial assistente.
* **`Windows Web Experience Pack`:** Remoção integral do pacote responsável por renderizar feeds e widgets flutuantes.
* **Lista de Bloqueio de Terceiros (Background Access):** Restringe mais de 30 identificadores UWP (incluindo *Spotify, WhatsApp, LinkedIn, Microsoft Teams, Office Hub e Xbox Overlays*) de consumirem ciclos de CPU enquanto fechados.

### 🌐 4. Políticas Locais do Navegador (Microsoft Edge Policies)
O script cria uma árvore de políticas rígidas diretamente em `HKLM:\SOFTWARE\Policies\Microsoft\Edge` para moldar o comportamento do navegador:
* **`BackgroundModeEnabled = 0`:** Impede de forma absoluta que extensões e aplicativos do Edge rodem em segundo plano após o fechamento da janela principal.
* **`SyncDisabled = 1`:** Desliga os motores de sincronização contínua de dados na nuvem corporativa.
* **`Edge3PSerpTelemetryEnabled = 0`:** Bloqueia o envio de telemetria gerada por pesquisas em buscadores de terceiros.

### 🧠 5. Subsistemas Avançados do Kernel
* **`psapi.dll` (Win32 API):** Chamada nativa ao método `EmptyWorkingSet`, forçando todas as aplicações ativas a devolverem imediatamente páginas de memória RAM física não utilizadas ao gerenciador do sistema.
* **`DISM` (Deployment Image Servicing and Management):** Executa comandos a nível de imagem online para desativar permanentemente o recurso **Recall** em versões modernas do Windows 11.


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

[📥 Baixar windows-debloat.ps1](/files/windows-debloat.ps1)

---
*credits:  [@lue93](https://github.com/lue93)*
