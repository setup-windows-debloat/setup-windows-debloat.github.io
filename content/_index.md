---
title: "🌌 Meu Espaço Dark Mode"
date: 2026-07-08
draft: false
---

# 🛸 Explorando o Universo com Hugo

Este é um exemplo prático de uma página inicial minimalista. O texto que você está lendo agora foi escrito em formato Markdown estruturado e estilizado dinamicamente pelo nosso tema escuro personalizado.

### 🛠️ Tecnologias Ativas no Projeto:
* **Hugo:** O motor de renderização estática mais veloz do planeta.
* **GitHub Pages:** Infraestrutura de hospedagem segura, global e gratuita.
* **CSS Customizado:** Variáveis otimizadas para o conforto visual dos seus olhos.

### 💻 Notas de Desenvolvimento:
Para modificar a aparência desta página, basta abrir o arquivo de configuração de estilos localizado em `layouts/index.html`. Você também pode adicionar trechos de código como `hugo server` para rodar o ambiente de testes localmente.

 
# 🐚 Script PowerShell

Script para deixar o Windows 10/11 mais "leve", consumindo menos RAM.

```powershell


# -------------------------------------------------------------------------------
# ETAPA 1: REMOÇÃO E BLOQUEIO DE TELEMETRIA, TRACKING E DIAGNÓSTICOS
# -------------------------------------------------------------------------------
Write-Host "[1/4] Expurgando serviços de Telemetria e Coleta de Dados..." -ForegroundColor Cyan

$TelemetryServices = @(
    "DiagTrack",          # Connected User Experiences and Telemetry
    "dmwappushservice",    # WAP Push Message Routing Service
    "WerSvc",             # Windows Error Reporting Service
    "wscsvc",             # Windows Security Center
    "wuauserv"            # Windows Update (Garante o fim do MoUsoCoreWorker)
)

foreach ($Service in $TelemetryServices) {
    if (Get-Service -Name $Service -ErrorAction SilentlyContinue) {
        Write-Host " -> Parando e desabilitando permanentemente: $Service" -ForegroundColor Yellow
        Stop-Service -Name $Service -Force -ErrorAction SilentlyContinue
        Set-Service -Name $Service -StartupType Disabled -ErrorAction SilentlyContinue
    }
}

# -------------------------------------------------------------------------------
# ETAPA 2: AJUSTE DE EFEITOS VISUAIS (TRANSFORMANDO EM WINDOWS CLÁSSICO)
# -------------------------------------------------------------------------------
Write-Host ""
Write-Host "[2/4] Desativando Animações, Sombras e Efeitos Visuais..." -ForegroundColor Cyan

# Define a política global de efeitos para "Melhor Desempenho"
$VisualFXPath = "HKCU:\Software\Microsoft\Windows\CurrentVersion\Explorer\VisualEffects"
if (-not (Test-Path $VisualFXPath)) { New-Item $VisualFXPath -Force | Out-Null }
Set-ItemProperty -Path $VisualFXPath -Name "VisualFXSetting" -Value 2

# Desativa transições de janelas, animações e suavizações visuais herdadas
$DesktopPath = "HKCU:\Control Panel\Desktop"
Set-ItemProperty -Path $DesktopPath -Name "UserPreferencesMask" -Value ([byte[]](0x90,0x12,0x01,0x80,0x10,0x00,0x00,0x00)) -ErrorAction SilentlyContinue
Set-ItemProperty -Path $DesktopPath -Name "MinAnimate" -Value 0 -ErrorAction SilentlyContinue

# Desativa efeitos de transparência do menu iniciar e barra de tarefas
$ThemesPath = "HKCU:\Software\Microsoft\Windows\CurrentVersion\Themes\Personalize"
if (Test-Path $ThemesPath) {
    Set-ItemProperty -Path $ThemesPath -Name "EnableTransparency" -Value 0 -ErrorAction SilentlyContinue
}

# -------------------------------------------------------------------------------
# ETAPA 3: LIMPEZA DE NOTIFICAÇÕES, PROCESSO DE WIDGETS E SERVIÇOS FANTASMAS
# -------------------------------------------------------------------------------
Write-Host ""
Write-Host "[3/4] Eliminando Centrais de Notificação, Widgets e Serviços Legados..." -ForegroundColor Cyan

# Desativa a Central de Notificações (Action Center) completamente da barra
$PolicyPath = "HKCU:\Software\Policies\Microsoft\Windows\Explorer"
if (-not (Test-Path $PolicyPath)) { New-Item $PolicyPath -Force | Out-Null }
Set-ItemProperty -Path $PolicyPath -Name "DisableNotificationCenter" -Value 1

# Remove sugestões, anúncios e dicas do sistema (Content Delivery Manager)
$ContentPath = "HKCU:\Software\Microsoft\Windows\CurrentVersion\ContentDeliveryManager"
if (Test-Path $ContentPath) {
    Set-ItemProperty -Path $ContentPath -Name "SoftLandingEnabled" -Value 0 -ErrorAction SilentlyContinue
    Set-ItemProperty -Path $ContentPath -Name "SubscribedContent-338389Enabled" -Value 0 -ErrorAction SilentlyContinue
    Set-ItemProperty -Path $ContentPath -Name "SubscribedContent-338388Enabled" -Value 0 -ErrorAction SilentlyContinue
    Set-ItemProperty -Path $ContentPath -Name "SubscribedContent-353696Enabled" -Value 0 -ErrorAction SilentlyContinue
    Set-ItemProperty -Path $ContentPath -Name "SystemPaneSuggestionsEnabled" -Value 0 -ErrorAction SilentlyContinue
}

# Desativa por completo a barra de Notícias e Interesses (Widgets de tempo/notícias)
$FeedsPath = "HKCU:\Software\Microsoft\Windows\CurrentVersion\Feeds"
if (-not (Test-Path $FeedsPath)) { New-Item $FeedsPath -Force | Out-Null }
Set-ItemProperty -Path $FeedsPath -Name "ShellFeedsTaskbarViewMode" -Value 2

# Desativa outros serviços secundários que rodam em background à toa
$LegacyServices = @("RemoteRegistry", "MapsBroker", "WbioSrvc", "Fax", "lmhosts", "wisvc")
foreach ($Service in $LegacyServices) {
    if (Get-Service -Name $Service -ErrorAction SilentlyContinue) {
        Stop-Service -Name $Service -Force -ErrorAction SilentlyContinue
        Set-Service -Name $Service -StartupType Disabled -ErrorAction SilentlyContinue
    }
}

# -------------------------------------------------------------------------------
# ETAPA 4: OTIMIZAÇÃO E LIMPEZA DE CACHE DO EXPLORADOR DE ARQUIVOS
# -------------------------------------------------------------------------------
Write-Host ""
Write-Host "[4/4] Bloqueando rastreamento de arquivos e anúncios no Explorer..." -ForegroundColor Cyan

$AdvancedPath = "HKCU:\Software\Microsoft\Windows\CurrentVersion\Advanced"
if (Test-Path $AdvancedPath) {
    # Desativa sugestões de provedores de sincronização (anúncios no explorer)
    Set-ItemProperty -Path $AdvancedPath -Name "ShowSyncProviderNotifications" -Value 0 -ErrorAction SilentlyContinue
    # Força o Explorer a abrir em "Este Computador" em vez do "Acesso Rápido"
    Set-ItemProperty -Path $AdvancedPath -Name "LaunchTo" -Value 1 -ErrorAction SilentlyContinue
}

# Desativa o histórico de arquivos e pastas frequentes para poupar indexação contínua de disco
$ExplorerPath = "HKCU:\Software\Microsoft\Windows\CurrentVersion\Explorer"
if (Test-Path $ExplorerPath) {
    Set-ItemProperty -Path $ExplorerPath -Name "ShowRecent" -Value 0 -ErrorAction SilentlyContinue
    Set-ItemProperty -Path $ExplorerPath -Name "ShowFrequent" -Value 0 -ErrorAction SilentlyContinue
}


Write-Host "Limpando e otimizando o Explorador de Arquivos..." -ForegroundColor Cyan

# Desativa anúncios/sugestões dentro do Explorador de Arquivos
$AdvancedPath = "HKCU:\Software\Microsoft\Windows\CurrentVersion\Advanced"
Set-ItemProperty -Path $AdvancedPath -Name "ShowSyncProviderNotifications" -Value 0

# Configura o Explorador para abrir direto em "Este Computador" em vez do Acesso Rápido
Set-ItemProperty -Path $AdvancedPath -Name "LaunchTo" -Value 1

# Desativa a exibição de arquivos e pastas abertos frequentemente (economiza indexação)
$ExplorerPath = "HKCU:\Software\Microsoft\Windows\CurrentVersion\Explorer"
Set-ItemProperty -Path $ExplorerPath -Name "ShowRecent" -Value 0
Set-ItemProperty -Path $ExplorerPath -Name "ShowFrequent" -Value 0


# -------------------------------------------------------------------------------
# ETAPA 5: PURGA DE SERVIÇOS DO CORE DO SISTEMA (SYSMAIN, DIAGNÓSTICOS, PESQUISA)
# -------------------------------------------------------------------------------
Write-Host "[1/5] Desativando Serviços de Telemetria Profunda, SysMain e Indexação..." -ForegroundColor Cyan

$HardcoreServices = @(
    "SysMain",            # Antigo Superfetch (Consome muita RAM criando cache invisível)
    "WSearch",            # Windows Search Indexer (Causa picos de I/O de disco no NVMe)
    "OneSyncSvc",         # Host de Sincronização da Microsoft (Contas, Email, Calendário)
    "PcaSvc",             # Assistente de Compatibilidade de Programas (Monitora execuções)
    "Troubleshooter",     # Serviço de Solução de Problemas do Windows
    "ShellHWDetection"    # Detecção de Hardware de Shell (Autoplay de mídias)
)

foreach ($Service in $HardcoreServices) {
    if (Get-Service -Name $Service -ErrorAction SilentlyContinue) {
        Write-Host " -> Eliminando Serviço Core: $Service" -ForegroundColor Yellow
        Stop-Service -Name $Service -Force -ErrorAction SilentlyContinue
        Set-Service -Name $Service -StartupType Disabled -ErrorAction SilentlyContinue
    }
}

# -------------------------------------------------------------------------------
# ETAPA 6: ARRASTANDO A CORTANA E A BUSCA DO BING PARA O LIMBO
# -------------------------------------------------------------------------------
Write-Host ""
Write-Host "[2/5] Destruindo Cortana, Telemetria do Bing e Integrações Cloud..." -ForegroundColor Cyan

$SearchPath = "HKCU:\Software\Microsoft\Windows\CurrentVersion\Search"
if (-not (Test-Path $SearchPath)) { New-Item $SearchPath -Force | Out-Null }
Set-ItemProperty -Path $SearchPath -Name "CortanaConsent" -Value 0
Set-ItemProperty -Path $SearchPath -Name "BingSearchEnabled" -Value 0
Set-ItemProperty -Path $SearchPath -Name "AllowSearchToUseLocation" -Value 0

$SearchPolicyPath = "HKLM:\SOFTWARE\Policies\Microsoft\Windows\Windows Search"
if (-not (Test-Path $SearchPolicyPath)) { New-Item $SearchPolicyPath -Force | Out-Null }
Set-ItemProperty -Path $SearchPolicyPath -Name "AllowCortana" -Value 0
Set-ItemProperty -Path $SearchPolicyPath -Name "DisableWebSearch" -Value 1
Set-ItemProperty -Path $SearchPolicyPath -Name "ConnectedSearchUseBing" -Value 0

# -------------------------------------------------------------------------------
# ETAPA 7: DESATIVANDO AGRESSIVAMENTE O SMARTSCREEN E RECURSOS DE TELEMETRIA OCULTOS
# -------------------------------------------------------------------------------
Write-Host ""
Write-Host "[3/5] Desativando SmartScreen (Verificação de arquivos) e Telemetria Oculta..." -ForegroundColor Cyan

# Desativa o SmartScreen para o Explorador de Arquivos e Aplicativos
$SmartScreenPath = "HKLM:\SOFTWARE\Microsoft\Windows\CurrentVersion\Explorer"
Set-ItemProperty -Path $SmartScreenPath -Name "SmartScreenEnabled" -Value "Off" -ErrorAction SilentlyContinue

# Desativa a gravação de passos, telemetria de digitação e histórico de atividade (Timeline)
$PrivacyPath = "HKLM:\SOFTWARE\Policies\Microsoft\Windows\DataCollection"
if (-not (Test-Path $PrivacyPath)) { New-Item $PrivacyPath -Force | Out-Null }
Set-ItemProperty -Path $PrivacyPath -Name "AllowTelemetry" -Value 0

$UserPrivacyPath = "HKCU:\Software\Microsoft\Windows\CurrentVersion\Privacy"
if (-not (Test-Path $UserPrivacyPath)) { New-Item $UserPrivacyPath -Force | Out-Null }
Set-ItemProperty -Path $UserPrivacyPath -Name "TailoredExperiencesWithDiagnosticDataEnabled" -Value 0

# -------------------------------------------------------------------------------
# ETAPA 8: MATANDO AGENDADORES DE TAREFAS OCULTOS DA MICROSOFT (TELEMETRIA DE DISCO)
# -------------------------------------------------------------------------------
Write-Host ""
Write-Host "[4/5] Desativando Tarefas Agendadas de Coleta de Dados e Customer Experience..." -ForegroundColor Cyan

$TasksToDisable = @(
    "\Microsoft\Windows\Application Experience\Microsoft Compatibility Appraiser",
    "\Microsoft\Windows\Application Experience\ProgramDataUpdater",
    "\Microsoft\Windows\Application Experience\StartupAppTask",
    "\Microsoft\Windows\Customer Experience Improvement Program\Consolidator",
    "\Microsoft\Windows\Customer Experience Improvement Program\Usbceip",
    "\Microsoft\Windows\DiskDiagnostic\Microsoft-Windows-DiskDiagnosticDataCollector",
    "\Microsoft\Windows\Maintenance\WinSAT" # Avaliação de desempenho automática em background
)

foreach ($Task in $TasksToDisable) {
    Disable-ScheduledTask -TaskName $Task -ErrorAction SilentlyContinue | Out-Null
    Write-Host " -> Tarefa Oculta Desativada: $Task" -ForegroundColor DarkYellow
}

# -------------------------------------------------------------------------------
# ETAPA 9: AJUSTES RESTRITOS NO GERENCIAMENTO DE MEMÓRIA (KERNEL)
# -------------------------------------------------------------------------------
Write-Host ""
Write-Host "[5/5] Forçando otimização de alocação de memória no Kernel do Windows..." -ForegroundColor Cyan

$MemoryManagementPath = "HKLM:\SYSTEM\CurrentControlSet\Control\Session Manager\Memory Management"
# Evita que o Windows crie paginações desnecessárias se houver RAM física disponível
Set-ItemProperty -Path $MemoryManagementPath -Name "DisablePagingExecutive" -Value 1 -ErrorAction SilentlyContinue
# Otimiza o tamanho do buffer de I/O para o sistema priorizar performance do sistema de arquivos
Set-ItemProperty -Path $MemoryManagementPath -Name "LargeSystemCache" -Value 0 -ErrorAction SilentlyContinue



# -------------------------------------------------------------------------------
# ETAPA 10: DESTRUIÇÃO DE SUBSISTEMAS PERSISTENTES (XBOX, GAME DVR, MAPS)
# -------------------------------------------------------------------------------

Write-Host "[1/4] Desativando Serviços de Background de Consumo Fantasma..." -ForegroundColor Cyan



$UltraServices = @(

    "XblAuthManager",      # Xbox Live Auth (Serviço de autenticação de jogos)

    "XblGameSave",         # Salve de jogos do Xbox

    "XboxNetApiSvc",       # Rede Xbox Live

    "B there",             # Serviços de geolocalização e mapas em background

    "lfsvc",               # Serviço de Geolocalização (Fica rastreando sua posição)

    "WpnService"           # Serviço de Notificações do Windows

)



foreach ($Service in $UltraServices) {

    if (Get-Service -Name $Service -ErrorAction SilentlyContinue) {

        Write-Host " -> Pulverizando Serviço: $Service" -ForegroundColor Yellow

        Stop-Service -Name $Service -Force -ErrorAction SilentlyContinue

        Set-Service -Name $Service -StartupType Disabled -ErrorAction SilentlyContinue

    }

}



# -------------------------------------------------------------------------------
# ETAPA 11: AJUSTES RESTRITOS DE REGISTRO PARA CORTE DE PROCESSOS REPETIDOS
# -------------------------------------------------------------------------------

Write-Host ""

Write-Host "[2/4] Configurando limites estritos para os processos de serviços (SvcHost)..." -ForegroundColor Cyan



# Força o Windows a agrupar processos idênticos do SvcHost em um único nó de RAM

$SvcHostPath = "HKLM:\SYSTEM\CurrentControlSet\Control"

Set-ItemProperty -Path $SvcHostPath -Name "SvcHostSplitThresholdInKB" -Value 3800000 -ErrorAction SilentlyContinue



# Desativa o GameDVR (Gravação em background da GameBar que consome RAM de vídeo/sistema)

$GameDVRPath = "HKCU:\System\GameConfigStore"

if (Test-Path $GameDVRPath) {

    Set-ItemProperty -Path $GameDVRPath -Name "GameDVR_Enabled" -Value 0 -ErrorAction SilentlyContinue

}

$GameDVRSystemPath = "HKLM:\SOFTWARE\Policies\Microsoft\Windows\GameDVR"

if (-not (Test-Path $GameDVRSystemPath)) { New-Item $GameDVRSystemPath -Force | Out-Null }

Set-ItemProperty -Path $GameDVRSystemPath -Name "AllowGameDVR" -Value 0 -ErrorAction SilentlyContinue



# -------------------------------------------------------------------------------
# ETAPA 12: VARREDURA DE PROCESSOS ZUMBIS DE TERCEIROS (TELEMETRIA ADICIONAL)
# -------------------------------------------------------------------------------

Write-Host ""

Write-Host "[3/4] Caçando e finalizando executáveis de telemetria ativa..." -ForegroundColor Cyan



$ZombieProcesses = @("MicrosoftEdgeUpdate", "GoogleUpdate", "IntelGraphicsCommandHandler", "igfxTray", "hkcmd")

foreach ($Proc in $ZombieProcesses) {

    if (Get-Process -Name $Proc -ErrorAction SilentlyContinue) {

        Write-Host " -> Derrubando processo zumbi: $Proc" -ForegroundColor Red

        Stop-Process -Name $Proc -Force -ErrorAction SilentlyContinue

    }

}




Write-Output "Desativando serviços de telemetria e diagnóstico..."

# --- DiagTrack (Connected User Experiences and Telemetry) ---
Write-Output "Desativando DiagTrack..."
Stop-Service DiagTrack -Force -ErrorAction SilentlyContinue
Set-Service DiagTrack -StartupType Disabled -ErrorAction SilentlyContinue

Write-Output "Desativando serviços opcionais..."

# --- Microsoft Edge Update ---
Write-Output "Desativando Microsoft Edge Update..."
Stop-Service edgeupdate -Force -ErrorAction SilentlyContinue
Set-Service edgeupdate -StartupType Disabled -ErrorAction SilentlyContinue
Stop-Service edgeupdatem -Force -ErrorAction SilentlyContinue
Set-Service edgeupdatem -StartupType Disabled -ErrorAction SilentlyContinue

# --- SearchApp (Busca do Windows) ---
Write-Output "Desativando SearchApp..."
Stop-Process -Name SearchApp -Force -ErrorAction SilentlyContinue
Set-ItemProperty -Path "HKCU:\Software\Microsoft\Windows\CurrentVersion\Search" -Name "BingSearchEnabled" -Value 0 -Force
Set-ItemProperty -Path "HKCU:\Software\Microsoft\Windows\CurrentVersion\Search" -Name "CortanaConsent" -Value 0 -Force

# --- PWA Helper ---
Write-Output "Desativando PWA Helper..."
Stop-Process -Name pwahelper -Force -ErrorAction SilentlyContinue




# -------------------------------------------------------------------------------
# ETAPA 14: API MEMORY FLUSH (FORÇAR COMPACTAÇÃO E DEVOLUÇÃO DE RAM AO SISTEMA)
# -------------------------------------------------------------------------------

Write-Host ""

Write-Host "[4/4] Executando chamada de API nativa para esvaziamento do Working Set..." -ForegroundColor Green



# Código C# injetado no PowerShell para acessar a API de Gerenciamento de Memória do Kernel NT

$MemoryFlushCode = @"

using System;

using System.Diagnostics;

using System.Runtime.InteropServices;



public class MemoryOptimizer {

    [DllImport("psapi.dll")]

    public static extern int EmptyWorkingSet(IntPtr hwProc);



    public static void Flush() {

        Process[] processes = Process.GetProcesses();

        foreach (Process p in processes) {

            try {

                EmptyWorkingSet(p.Handle);

            } catch { }

        }

    }

}

"@



Add-Type -TypeDefinition $MemoryFlushCode

[MemoryOptimizer]::Flush()


Write-Host " -> Memória física esvaziada e higienizada com sucesso!" -ForegroundColor Green


```
---
*Página gerada estaticamente com foco em alta performance e legibilidade noturna.*
