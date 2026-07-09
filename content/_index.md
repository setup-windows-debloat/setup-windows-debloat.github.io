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


## 🐚 Script PowerShell

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



# Desativa o Recall (Windows 11 24H2 ou superior)
DISM /Online /Disable-Feature /FeatureName:"Recall"

# Desativa o histórico da área de transferência
Set-ItemProperty -Path "HKCU:\Software\Microsoft\Clipboard" -Name "EnableClipboardHistory" -Value 0

# Desativa a sincronização da área de transferência entre dispositivos
Set-ItemProperty -Path "HKCU:\Software\Microsoft\Clipboard" -Name "EnableSync" -Value 0

# Desativa a Otimização de Entrega
Stop-Service -Name "DoSvc"
Set-Service -Name "DoSvc" -StartupType Disabled

# Desativa o serviço de Pesquisa do Windows
Stop-Service -Name "WSearch"
Set-Service -Name "WSearch" -StartupType Disabled

# Desativa Área de Trabalho Remota
Stop-Service -Name "TermService"
Set-Service -Name "TermService" -StartupType Disabled

# Desativa o SysMain (Superfetch) – útil especialmente em SSDs
Stop-Service -Name "SysMain"
Set-Service -Name "SysMain" -StartupType Disabled

# Desativa permanentemente o Defender 
# Para ativar use -> Set-MpPreference -DisableRealtimeMonitoring $false 
New-ItemProperty -Path "HKLM:\SOFTWARE\Policies\Microsoft\Windows Defender" -Name "DisableAntiSpyware" -Value 1 -PropertyType DWORD -Force
New-ItemProperty -Path "HKLM:\SOFTWARE\Policies\Microsoft\Windows Defender" -Name "DisableAntiVirus" -Value 1 -PropertyType DWORD -Force
New-ItemProperty -Path "HKLM:\SOFTWARE\Policies\Microsoft\Windows Defender" -Name "ServiceKeepAlive" -Value 0 -PropertyType DWORD -Force


# 🧼 Desativa a sincronização da área de transferência (Clipboard na nuvem)
New-ItemProperty -Path "HKCU:\Software\Microsoft\Clipboard" ` -Name "CloudClipboardDisabled" -Value 1 -PropertyType DWORD -Force


Set-MpPreference -DisableRealtimeMonitoring $true
Set-MpPreference -DisableBehaviorMonitoring $true
Set-MpPreference -DisableBlockAtFirstSeen $true
Set-MpPreference -DisableIOAVProtection $true
Set-MpPreference -DisablePrivacyMode $true
Set-MpPreference -DisableScriptScanning $true

New-Item -Path "HKLM:\SOFTWARE\Policies\Microsoft\Edge" -Force
Set-ItemProperty -Path "HKLM:\SOFTWARE\Policies\Microsoft\Edge" -Name "SyncDisabled" -Value 1

Get-AppxPackage Microsoft.YourPhone -AllUsers | Remove-AppxPackage

Set-ItemProperty -Path "HKCU:\Software\Microsoft\Windows\CurrentVersion\BackgroundAccessApplications\Microsoft.Copilot_8wekyb3d8bbwe" -Name "Disabled" -Value 1

Get-ChildItem -Path "HKCU:\Software\Microsoft\Windows\CurrentVersion\BackgroundAccessApplications" | Select-Object Name

# Lista de AppIDs a serem desativados
$apps = @(
    "7EE7776C.LinkedInforWindows_w1wdnht996qgy",
    "43891JeniusApps.NightingaleRESTAPIClient_jaj7tphbgjeh8",
    "B0A091E8.CLIPSTUDIOPAINTSTARTforSC_2vfvh2vszw3jw",
    "app.diagrams.net-F22DFE3B_1nhq2b0gx0f14",
    "5319275A.WhatsAppDesktop_cv1g1gvanyjgm",
    "excalidraw.com-A97B1CC3_v2mdq74wtmey8",
    "github.com-8B11BEB2_2t1n1bqhyggy0",
    "Microsoft.BingSearch_8wekyb3d8bbwe",
    "Microsoft.BingWallpaper_8wekyb3d8bbwe",
    "Microsoft.Edge.GameAssist_8wekyb3d8bbwe",
    "Microsoft.GamingApp_8wekyb3d8bbwe",
    "Microsoft.MicrosoftOfficeHub_8wekyb3d8bbwe",
    "Microsoft.MicrosoftSolitaireCollection_8wekyb3d8bbwe",
    "Microsoft.MicrosoftStickyNotes_8wekyb3d8bbwe",
    "Microsoft.OneDriveSync_8wekyb3d8bbwe",
    "Microsoft.OutlookForWindows_8wekyb3d8bbwe",
    "Microsoft.Whiteboard_8wekyb3d8bbwe",
    "Microsoft.Xbox.TCUI_8wekyb3d8bbwe",
    "Microsoft.XboxGameCallableUI_cw5n1h2txyewy",
    "Microsoft.XboxGameOverlay_8wekyb3d8bbwe",
    "Microsoft.XboxGamingOverlay_8wekyb3d8bbwe",
    "Microsoft.XboxIdentityProvider_8wekyb3d8bbwe",
    "Microsoft.XboxSpeechToTextOverlay_8wekyb3d8bbwe",
    "MicrosoftTeams_8wekyb3d8bbwe",
    "MicrosoftWindows.CrossDevice_cw5n1h2txyewy",
    "MSTeams_8wekyb3d8bbwe",
    "Sidia.LiveWallpaper_wkpx6gdq8qyz8",
    "SpotifyAB.SpotifyMusic_zpdnekdrzrea0",
    "www.msn.com-C9B6222F_q77jw2zwjvy92",
    "www.youtube.com-54E21B02_pd8mbgmqs65xy"
)

# Caminho base do Registro
$basePath = "HKCU:\Software\Microsoft\Windows\CurrentVersion\BackgroundAccessApplications"

# Desativa cada aplicativo
foreach ($app in $apps) {
    $fullPath = Join-Path -Path $basePath -ChildPath $app
    try {
        Set-ItemProperty -Path $fullPath -Name "Disabled" -Value 1 -ErrorAction Stop
        Write-Host "Desativado com sucesso: $app"
    } catch {
        Write-Warning "Não foi possível modificar: $app — $_"
    }
}

# Caminho do registro
$regPath = "HKLM:\SOFTWARE\Policies\Microsoft\Edge"

# Cria a chave se não existir
if (-not (Test-Path $regPath)) {
    New-Item -Path $regPath -Force | Out-Null
}

# Define os valores para impedir execução em segundo plano
New-ItemProperty -Path $regPath -Name "AllowPrelaunch" -Value 0 -PropertyType DWord -Force
New-ItemProperty -Path $regPath -Name "AllowBackgroundProcess" -Value 0 -PropertyType DWord -Force

Write-Output "Configurações aplicadas: Edge não será executado em segundo plano."

# Caminho do registro
$regPath = "HKCU:\Software\Microsoft\Windows\CurrentVersion\CrossDeviceResume\Configuration"

# Cria a chave se não existir
if (-not (Test-Path $regPath)) {
    New-Item -Path $regPath -Force | Out-Null
}

# Desativa o recurso
New-ItemProperty -Path $regPath -Name "IsResumeAllowed" -Value 0 -PropertyType DWord -Force

Write-Output "CrossDeviceResume desativado com sucesso."


# Filtra todos os processos que estão usando msedgewebview2.exe ou têm referência ao WebView2
Get-Process | Where-Object {
    $_.Path -like "*msedgewebview2.exe*" -or
    ($_.Modules | Where-Object { $_.FileName -like "*WebView2*" })
} | Select-Object Name, Id, Path | Format-Table -AutoSize


# Verifica se o pacote está instalado
$package = Get-AppxPackage -Name "*WebExperience*"
if ($package) {
    Write-Host "Removendo o Windows Web Experience Pack..."
    Remove-AppxPackage -Package $package.PackageFullName
    Write-Host "Widgets desabilitados com sucesso."
} else {
    Write-Host "O pacote WebExperience já foi removido ou não está instalado."
}


# Definir a política para bloquear a execução em segundo plano da Microsoft Store
$appPackage = "Microsoft.WindowsStore"
Get-AppxPackage -Name $appPackage | ForEach-Object {
    $packageFamilyName = $_.PackageFamilyName
    $registryPath = "HKCU:\Software\Microsoft\Windows\CurrentVersion\BackgroundAccessApplications\$packageFamilyName"
    
    New-Item -Path $registryPath -Force | Out-Null
    Set-ItemProperty -Path $registryPath -Name "Disabled" -Value 1
    Write-Output "Execução em segundo plano desabilitada para Microsoft Store."
}

$servicos = @(
    "BITS",         # Transferência em segundo plano (usado pelo Windows Update e possivelmente Bing)
    "DiagTrack",    # Diagnóstico de compatibilidade e telemetria
    "WSearch",      # Indexação de arquivos (pode afetar buscas locais)
    "SysMain",      # Superfetch (pré-carregamento de apps)
    "XblGameSave",  # Serviços Xbox
    "XboxNetApiSvc" # Serviços Xbox
)

foreach ($s in $servicos) {
    try {
        Stop-Service -Name $s -Force -ErrorAction SilentlyContinue
        Set-Service -Name $s -StartupType Disabled
        Write-Host "Desativado: $s"
    } catch {
        Write-Warning "Falha ao desativar $s"
    }
}


# Lista de serviços relacionados à telemetria
$servicos = @(
    "DiagTrack",           # Experiências do Usuário e Telemetria
    "dmwappushservice",    # Push de dados de diagnóstico
    "WerSvc"               # Serviço de Relatório de Erros
)

foreach ($s in $servicos) {
    try {
        Stop-Service -Name $s -Force -ErrorAction SilentlyContinue
        Set-Service -Name $s -StartupType Disabled
        Write-Host "Desativado serviço: $s"
    } catch {
        Write-Warning "Falha ao desativar serviço: $s"
    }
}

# Desativar tarefas agendadas de telemetria
$tasks = @(
    "\Microsoft\Windows\Application Experience\Microsoft Compatibility Appraiser",
    "\Microsoft\Windows\Customer Experience Improvement Program\Consolidator",
    "\Microsoft\Windows\Customer Experience Improvement Program\UsbCeip",
    "\Microsoft\Windows\Autochk\Proxy"
)

foreach ($task in $tasks) {
    try {
        schtasks /Change /TN $task /DISABLE
        Write-Host "Desativada tarefa: $task"
    } catch {
        Write-Warning "Falha ao desativar tarefa: $task"
    }
}

# Ajuste no registro para desativar telemetria
try {
    New-Item -Path "HKLM:\SOFTWARE\Policies\Microsoft\Windows\DataCollection" -Force | Out-Null
    New-ItemProperty -Path "HKLM:\SOFTWARE\Policies\Microsoft\Windows\DataCollection" -Name "AllowTelemetry" -PropertyType DWord -Value 0 -Force | Out-Null
    Write-Host "Registro atualizado: AllowTelemetry = 0"
} catch {
    Write-Warning "Falha ao atualizar o registro"
}


# Verifica entradas de inicialização
Get-ItemProperty -Path "HKLM:\Software\Microsoft\Windows\CurrentVersion\Run" |
    Select-Object IntelConnect

Get-ItemProperty -Path "HKCU:\Software\Microsoft\Windows\CurrentVersion\Run" |
    Select-Object IntelConnect

# Remove do Registro (LocalMachine ou CurrentUser)
Remove-ItemProperty -Path "HKLM:\Software\Microsoft\Windows\CurrentVersion\Run" -Name "IntelConnect" -ErrorAction SilentlyContinue
Remove-ItemProperty -Path "HKCU:\Software\Microsoft\Windows\CurrentVersion\Run" -Name "IntelConnect" -ErrorAction SilentlyContinue

# Cria ou altera a chave de registro para desativar sugestões de pesquisa
$registryPath = "HKCU:\Software\Policies\Microsoft\Windows\Explorer"
If (-Not (Test-Path $registryPath)) {
    New-Item -Path $registryPath -Force | Out-Null
}
New-ItemProperty -Path $registryPath -Name "DisableSearchBoxSuggestions" -PropertyType DWord -Value 1 -Force

Write-Host "Sugestões de pesquisa desativadas com sucesso."



# Desativa o serviço de indexação (Windows Search)
Get-Service -Name "WSearch" | Set-Service -StartupType Disabled
Stop-Service -Name "WSearch" -Force

# Remove a barra de pesquisa da barra de tarefas (0 = Oculto)
Set-ItemProperty -Path "HKCU:\Software\Microsoft\Windows\CurrentVersion\Search" -Name "SearchboxTaskbarMode" -Value 0

# Desativa sugestões da web no menu iniciar
Set-ItemProperty -Path "HKCU:\Software\Policies\Microsoft\Windows\Explorer" -Name "DisableSearchBoxSuggestions" -Value 1

# Desativa histórico de pesquisa
Set-ItemProperty -Path "HKCU:\Software\Microsoft\Windows\CurrentVersion\SearchSettings" -Name "IsDeviceSearchHistoryEnabled" -Value 0

# Reinicia o Explorer para aplicar mudanças visuais
Stop-Process -Name explorer -Force
Start-Process explorer

Write-Host "Recursos de pesquisa desativados com sucesso."

$services = @(
    "DiagTrack",       # Serviço de rastreamento de diagnóstico (telemetria)
    "dmwappushsvc",    # Serviço de envio de dados de diagnóstico
    "MapsBroker",      # Cache de mapas offline
    "SysMain",         # Superfetch (pré-carregamento de apps)
    "WSearch",         # Indexação de arquivos
    "BITS"             # Transferência em segundo plano (usado pelo Windows Update)
)

foreach ($s in $services) {
    try {
        Stop-Service -Name $s -Force -ErrorAction SilentlyContinue
        Set-Service -Name $s -StartupType Disabled
        Write-Host "Desativado: $s"
    } catch {
        Write-Warning "Falha ao desativar $s"
    }
}

# Desativa o SmartScreen via Registro
$regPath = "HKLM:\SOFTWARE\Policies\Microsoft\Windows\System"
New-Item -Path $regPath -Force | Out-Null
New-ItemProperty -Path $regPath -Name "EnableSmartScreen" -Value 0 -PropertyType DWORD -Force

# Opcional: define o nível de bloqueio (Avisar ou Bloquear)
New-ItemProperty -Path $regPath -Name "ShellSmartScreenLevel" -Value "Warn" -PropertyType String -Force

Write-Host "SmartScreen desativado via Registro."


# Lista dos serviços a serem desativados
$services = @("UsoSvc", "wuauserv", "WinDefend", "DolbyDAXAPI", "DoSvc")

foreach ($s in $services) {
    try {
        # Tenta parar e desativar o serviço
        Stop-Service -Name $s -Force -ErrorAction SilentlyContinue
        Set-Service -Name $s -StartupType Disabled
        Write-Host "✅ Serviço desativado: $s"
    } catch {
        Write-Warning "⚠️ Não foi possível desativar: $s"
    }
}


# Desativar o serviço Windows Mobile Hotspot
Set-Service -Name icssvc -StartupType Disabled
Stop-Service -Name icssvc -Force

# Desativar o Serviço de Demonstração de Varejo
Set-Service -Name RetailDemo -StartupType Disabled
Stop-Service -Name RetailDemo -Force



# Criar chave de políticas do Edge
New-Item -Path "HKLM:\SOFTWARE\Policies\Microsoft\Edge" -Force

# 🔧 Desempenho: desativar Sleeping Tabs (se preferir manter abas ativas)
New-ItemProperty -Path "HKLM:\SOFTWARE\Policies\Microsoft\Edge" -Name "SleepingTabsEnabled" -Value 0 -PropertyType DWord -Force

# 🔒 Privacidade: bloquear WebWidget (busca flutuante)
New-ItemProperty -Path "HKLM:\SOFTWARE\Policies\Microsoft\Edge" -Name "WebWidgetAllowed" -Value 0 -PropertyType DWord -Force

# 🔒 Privacidade: desativar conteúdo promocional
New-ItemProperty -Path "HKLM:\SOFTWARE\Policies\Microsoft\Edge" -Name "PromotionalTabsEnabled" -Value 0 -PropertyType DWord -Force

# 🔧 Desempenho: impedir execução de extensões em segundo plano
New-ItemProperty -Path "HKLM:\SOFTWARE\Policies\Microsoft\Edge" -Name "BackgroundModeEnabled" -Value 0 -PropertyType DWord -Force

# 🔒 Privacidade: desativar compartilhamento de dados de navegação
New-ItemProperty -Path "HKLM:\SOFTWARE\Policies\Microsoft\Edge" -Name "LocalBrowserDataShareEnabled" -Value 0 -PropertyType DWord -Force

# 🔒 Privacidade: desativar sincronização automática
New-ItemProperty -Path "HKLM:\SOFTWARE\Policies\Microsoft\Edge" -Name "SyncDisabled" -Value 1 -PropertyType DWord -Force

# 🔒 Privacidade: ativar modo de rastreamento rigoroso
New-ItemProperty -Path "HKLM:\SOFTWARE\Policies\Microsoft\Edge" -Name "TrackingPrevention" -Value 2 -PropertyType DWord -Force


# Criar chave de políticas do Edge
New-Item -Path "HKLM:\SOFTWARE\Policies\Microsoft\Edge" -Force

# 🔒 Desativar Telemetria SERP (pesquisas em buscadores de terceiros)
New-ItemProperty -Path "HKLM:\SOFTWARE\Policies\Microsoft\Edge" -Name "Edge3PSerpTelemetryEnabled" -Value 0 -PropertyType DWord -Force

# 🔒 Desativar envio de dados de diagnóstico opcionais
New-ItemProperty -Path "HKLM:\SOFTWARE\Policies\Microsoft\Edge" -Name "DiagnosticData" -Value 1 -PropertyType DWord -Force

# 🔒 Desativar compartilhamento de dados locais do navegador
New-ItemProperty -Path "HKLM:\SOFTWARE\Policies\Microsoft\Edge" -Name "LocalBrowserDataShareEnabled" -Value 0 -PropertyType DWord -Force

# 🔒 Desativar sincronização de dados com a nuvem
New-ItemProperty -Path "HKLM:\SOFTWARE\Policies\Microsoft\Edge" -Name "SyncDisabled" -Value 1 -PropertyType DWord -Force

# 🔒 Ativar proteção contra rastreamento no modo rigoroso
New-ItemProperty -Path "HKLM:\SOFTWARE\Policies\Microsoft\Edge" -Name "TrackingPrevention" -Value 2 -PropertyType DWord -Force

# Criar chave de políticas
New-Item -Path "HKLM:\SOFTWARE\Policies\Microsoft\Windows\DataCollection" -Force

# 🔒 Desativar telemetria
New-ItemProperty -Path "HKLM:\SOFTWARE\Policies\Microsoft\Windows\DataCollection" -Name "AllowTelemetry" -Value 0 -PropertyType DWord -Force

# 🔒 Desativar envio de dados de diagnóstico
New-ItemProperty -Path "HKLM:\SOFTWARE\Policies\Microsoft\Windows\DataCollection" -Name "SendYourDataToMicrosoft" -Value 0 -PropertyType DWord -Force

# 🔒 Desativar compartilhamento de dados locais
New-ItemProperty -Path "HKLM:\SOFTWARE\Policies\Microsoft\Edge" -Name "LocalBrowserDataShareEnabled" -Value 0 -PropertyType DWord -Force

# 🔒 Desativar sincronização de dados
New-ItemProperty -Path "HKLM:\SOFTWARE\Policies\Microsoft\Edge" -Name "SyncDisabled" -Value 1 -PropertyType DWord -Force

# 🔒 Desativar rastreadores de publicidade
New-ItemProperty -Path "HKCU:\Software\Microsoft\Windows\CurrentVersion\AdvertisingInfo" -Name "Enabled" -Value 0 -PropertyType DWord -Force

# 🔒 Desativar sugestões e anúncios no menu Iniciar
New-ItemProperty -Path "HKCU:\Software\Microsoft\Windows\CurrentVersion\ContentDeliveryManager" -Name "SubscribedContent-338388Enabled" -Value 0 -PropertyType DWord -Force
New-ItemProperty -Path "HKCU:\Software\Microsoft\Windows\CurrentVersion\ContentDeliveryManager" -Name "SystemPaneSuggestionsEnabled" -Value 0 -PropertyType DWord -Force

# 🔒 Desativar anúncios na tela de bloqueio
New-ItemProperty -Path "HKCU:\Software\Microsoft\Windows\CurrentVersion\ContentDeliveryManager" -Name "RotatingLockScreenEnabled" -Value 0 -PropertyType DWord -Force
New-ItemProperty -Path "HKCU:\Software\Microsoft\Windows\CurrentVersion\ContentDeliveryManager" -Name "RotatingLockScreenOverlayEnabled" -Value 0 -PropertyType DWord -Force

# 🔒 Desativar KYC (identificação de usuário via conta Microsoft)
New-ItemProperty -Path "HKLM:\SOFTWARE\Policies\Microsoft\Windows\System" -Name "EnableSmartScreen" -Value 0 -PropertyType DWord -Force
New-ItemProperty -Path "HKLM:\SOFTWARE\Policies\Microsoft\Windows\System" -Name "EnableWebContentEvaluation" -Value 0 -PropertyType DWord -Force

# Desativar tarefas de rastreio
$tasks = @(
  "\Microsoft\Windows\Application Experience\ProgramDataUpdater",
  "\Microsoft\Windows\Autochk\Proxy",
  "\Microsoft\Windows\Customer Experience Improvement Program\Consolidator",
  "\Microsoft\Windows\Customer Experience Improvement Program\UsbCeip",
  "\Microsoft\Windows\DiskDiagnostic\Microsoft-Windows-DiskDiagnosticDataCollector"
)

foreach ($task in $tasks) {
  schtasks /Change /TN $task /Disable
}

# Remover apps desnecessários
$apps = @(
  "Microsoft.YourPhone",
  "Microsoft.XboxGameOverlay",
  "Microsoft.XboxGamingOverlay",
  "Microsoft.People",
  "Microsoft.MicrosoftSolitaireCollection",
  "Microsoft.BingNews",
  "Microsoft.GetHelp"
)

foreach ($app in $apps) {
  Get-AppxPackage -Name $app | Remove-AppxPackage
}

# Desabilita o recurso "Restart Apps" no logon
$registryPath = "HKCU:\Software\Microsoft\Windows\CurrentVersion\Explorer\RestartApps"
New-Item -Path $registryPath -Force | Out-Null
Set-ItemProperty -Path $registryPath -Name "Disabled" -Value 1


# Verifica se o serviço WSAIFabricSvc existe
$servico = Get-Service -Name "WSAIFabricSvc" -ErrorAction SilentlyContinue

if ($servico) {
    Write-Output "Desativando o serviço WSAIFabricSvc..."

    # Para o serviço, se estiver em execução
    if ($servico.Status -eq "Running") {
        Stop-Service -Name "WSAIFabricSvc" -Force
    }

    # Define o tipo de inicialização como desabilitado
    Set-Service -Name "WSAIFabricSvc" -StartupType Disabled

    Write-Output "Serviço desativado com sucesso."
} else {
    Write-Output "Serviço WSAIFabricSvc não encontrado no sistema."
}

Set-MpPreference -DisableRealtimeMonitoring $true
Set-MpPreference -DisableBehaviorMonitoring $true
Set-MpPreference -DisableBlockAtFirstSeen $true
Set-MpPreference -DisableIOAVProtection $true
Set-MpPreference -DisablePrivacyMode $true
Set-MpPreference -SignatureDisableUpdateOnStartupWithoutEngine $true

New-ItemProperty -Path "HKLM:\Software\Policies\Microsoft\Windows Defender" `
  -Name DisableAntiSpyware -Value 1 -PropertyType DWORD -Force

# Lista de serviços a desativar
$servicos = @(
    "PenService_7affd",
    "PhoneSvc",
    "OneSyncSvc_7affd",
    "ClipSVC",
    "CDPUserSvc_7affd",
    "UdkUserSvc_7affd",
    "WpnUserService_7affd"
)

foreach ($servico in $servicos) {
    Write-Host "Desativando serviço: $servico"
    
    # Parar o serviço se estiver em execução
    Stop-Service -Name $servico -ErrorAction SilentlyContinue

    # Definir tipo de inicialização como desativado
    Set-Service -Name $servico -StartupType Disabled -ErrorAction SilentlyContinue
}

Write-Host "Todos os serviços foram desativados com sucesso."


Stop-Service -Name wuauserv
Set-Service -Name wuauserv -StartupType Disabled



# Desativa proteção em tempo real
Set-MpPreference -DisableRealtimeMonitoring $true

# Bloqueia execução em segundo plano via registro
New-Item -Path "HKLM:\SOFTWARE\Policies\Microsoft\Windows\AppPrivacy" -Force | Out-Null
New-ItemProperty -Path "HKLM:\SOFTWARE\Policies\Microsoft\Windows\AppPrivacy" `
    -Name "LetAppsRunInBackground" -PropertyType DWord -Value 2 -Force | Out-Null

# Bloqueia MDCoreSvc via Image File Execution Options
$regPath = "HKLM:\SOFTWARE\Microsoft\Windows NT\CurrentVersion\Image File Execution Options\MpDefenderCoreService.exe"
New-Item -Path $regPath -Force | Out-Null
New-ItemProperty -Path $regPath -Name "Debugger" -Value "%SystemRoot%\System32\taskkill.exe" -PropertyType String -Force | Out-Null

Write-Host "✅ Serviços bloqueados e execução em segundo plano desativada. Reinicie o sistema para aplicar totalmente."

# Exibe todas as propriedades da chave WinDefend
Get-ItemProperty -Path "HKLM:\SYSTEM\CurrentControlSet\Services\WinDefend" | Format-List

Set-ItemProperty -Path "HKLM:\SYSTEM\CurrentControlSet\Services\WinDefend" -Name "Start" -Value 4

Get-Service | Where-Object {$_.Status -eq 'Running'} | Select-Object Name, DisplayName, Status | Format-Table -AutoSize


Get-AppxPackage | ForEach-Object {
    $packageName = $_.PackageFullName
    $appId = $_.Name
    Try {
        $settingsPath = "HKCU:\Software\Microsoft\Windows\CurrentVersion\BackgroundAccessApplications\$packageName"
        If (Test-Path $settingsPath) {
            Set-ItemProperty -Path $settingsPath -Name "Disabled" -Value 1
            Write-Host "Execução em segundo plano desativada para: $appId"
        }
    } Catch {
        Write-Host "Erro ao processar: $appId"
    }
}

Set-Service -Name "SysMain" -StartupType Disabled   # Superfetch: inútil em SSDs
Set-Service -Name "WSearch" -StartupType Disabled   # Indexação de arquivos
Set-Service -Name "DiagTrack" -StartupType Disabled # Telemetria
Set-Service -Name "Fax" -StartupType Disabled       # Serviço de fax (quem usa?)

Remove-Item -Path "$env:TEMP\*" -Recurse -Force
Remove-Item -Path "C:\Windows\Temp\*" -Recurse -Force
Clear-RecycleBin -Force

Get-AppxPackage -AllUsers | Where-Object {
    $_.Name -like "*Xbox*" -or
    $_.Name -like "*Skype*" -or
    $_.Name -like "*Solitaire*" -or
    $_.Name -like "*People*" -or
    $_.Name -like "*Weather*"
} | Remove-AppxPackage


Get-WindowsOptionalFeature -Online | Where-Object {
    $_.FeatureName -like "*Media*" -or
    $_.FeatureName -like "*XPS*" -or
    $_.FeatureName -like "*WorkFolders*"
} | ForEach-Object {
    Disable-WindowsOptionalFeature -Online -FeatureName $_.FeatureName -NoRestart
}

New-ItemProperty -Path "HKCU:\Software\Microsoft\Windows\CurrentVersion\ContentDeliveryManager" -Name "SystemPaneSuggestionsEnabled" -PropertyType DWord -Value 0 -Force

Set-MpPreference -DisableRealtimeMonitoring $true
New-ItemProperty -Path "HKLM:\Software\Policies\Microsoft\Windows Defender" -Name "DisableAntiSpyware" -Value 1 -PropertyType DWORD -Force
Set-MpPreference -ScanAvgCPULoadFactor 2
Set-MpPreference -DisableScheduledScans $true
Set-MpPreference -DisableBehaviorMonitoring $true
Set-MpPreference -DisableArchiveScanning $true
Set-MpPreference -DisableEmailScanning $true
Set-MpPreference -DisableIntrusionPreventionSystem $true
Set-MpPreference -DisableIOAVProtection $true


Set-MpPreference -DisableIdleScans $true
Set-MpPreference -MAPSReporting Disabled
Set-MpPreference -SubmitSamplesConsent NeverSend
Set-MpPreference -DisableBehaviorMonitoring $true

# Desativar sugestões e conteúdo promocional
Set-ItemProperty -Path "HKCU:\Software\Microsoft\Windows\CurrentVersion\ContentDeliveryManager" -Name "SubscribedContent-310093Enabled" -Value 0
Set-ItemProperty -Path "HKCU:\Software\Microsoft\Windows\CurrentVersion\ContentDeliveryManager" -Name "RotatingLockScreenEnabled" -Value 0
Set-ItemProperty -Path "HKCU:\Software\Microsoft\Windows\CurrentVersion\ContentDeliveryManager" -Name "RotatingLockScreenOverlayEnabled" -Value 0


# Desabilita sugestões de pesquisa recentes no Explorador de Arquivos
$registryPath = "HKCU:\Software\Policies\Microsoft\Windows\Explorer"
$registryName = "DisableSearchBoxSuggestions"

# Cria a chave se não existir
if (-not (Test-Path $registryPath)) {
    New-Item -Path $registryPath -Force | Out-Null
}

# Define o valor para desativar sugestões
Set-ItemProperty -Path $registryPath -Name $registryName -Value 1 -Type DWord

Write-Host "Sugestões de pesquisa desativadas com sucesso. Reinicie o computador para aplicar."

# Remove apps comuns que vêm com o Windows
$apps = @(
    "Microsoft.YourPhone",
    "Microsoft.XboxGamingOverlay",
    "Microsoft.GetHelp",
    "Microsoft.People",
    "Microsoft.MicrosoftSolitaireCollection",
    "Microsoft.BingNews",
    "Microsoft.WindowsMaps"
)

foreach ($app in $apps) {
    Get-AppxPackage -Name $app | Remove-AppxPackage
}

# Desativa serviços de telemetria
Set-ItemProperty -Path "HKLM:\SOFTWARE\Policies\Microsoft\Windows\DataCollection" -Name "AllowTelemetry" -Value 0 -Type DWord


# Remove sugestões de apps e notificações
Set-ItemProperty -Path "HKCU:\Software\Microsoft\Windows\CurrentVersion\ContentDeliveryManager" -Name "SubscribedContent-338388Enabled" -Value 0
Set-ItemProperty -Path "HKCU:\Software\Microsoft\Windows\CurrentVersion\ContentDeliveryManager" -Name "SubscribedContent-310093Enabled" -Value 0


# Lista e remove programas de inicialização
Get-CimInstance Win32_StartupCommand | ForEach-Object {
    if ($_.Location -ne $null) {
        Write-Host "Removendo: $($_.Name)"
        Remove-Item -Path $_.Location -Force
    }
}


# Desativa serviços que consomem recursos
$services = @("DiagTrack", "WSearch", "SysMain", "Fax")

foreach ($svc in $services) {
    Stop-Service -Name $svc -Force
    Set-Service -Name $svc -StartupType Disabled
}


# Desativa Cortana e busca online
Set-ItemProperty -Path "HKCU:\Software\Microsoft\Windows\CurrentVersion\Search" -Name "CortanaConsent" -Value 0
Set-ItemProperty -Path "HKCU:\Software\Microsoft\Windows\CurrentVersion\Search" -Name "SearchOnTaskbarMode" -Value 0


Get-Process | Where-Object { $_.MainWindowHandle -eq 0 } | Sort-Object WorkingSet -Descending | Select-Object Name, Id, CPU, @{Name="MemoryMB";Expression={[math]::round($_.WorkingSet / 1MB, 2)}}

# Verifica se o serviço de pesquisa do Windows existe
$servico = Get-Service -Name "WSearch" -ErrorAction SilentlyContinue

if ($servico) {
    # Para o serviço se estiver em execução
    if ($servico.Status -eq "Running") {
        Stop-Service -Name "WSearch" -Force
        Write-Host "Serviço de pesquisa parado." -ForegroundColor Yellow
    }

    # Desativa o serviço para não iniciar com o sistema
    Set-Service -Name "WSearch" -StartupType Disabled
    Write-Host "Serviço de pesquisa desativado. SearchHost não será iniciado." -ForegroundColor Green
} else {
    Write-Host "Serviço 'WSearch' não encontrado. Pode já estar desativado ou ausente." -ForegroundColor Red
}


powercfg -setactive SCHEME_MIN
Write-Host "Plano de energia 'Alto desempenho' ativado." -ForegroundColor Cyan


# Nome do processo do Copilot (ajuste se necessário)
$processName = "Copilot"

# Verifica se o processo está em execução
$process = Get-Process -Name $processName -ErrorAction SilentlyContinue

if ($process) {
    Write-Host "Encerrando o processo '$processName'..."
    Stop-Process -Name $processName -Force
    Write-Host "Processo encerrado com sucesso."
} else {
    Write-Host "O processo '$processName' não está em execução."
}


$process = Get-Process -Name "SearchHost"
$process.PriorityClass = "BelowNormal"



```
---

# 📥 Área de Downloads

Clique no botão abaixo para baixar o script operacional de automação diretamente para a sua máquina:

[📥 Baixar Script PowerShell](/arquivos/meu-script.ps1){.btn-download download="meu-script.ps1"}

---
*Página gerada estaticamente com foco em alta performance e legibilidade noturna.*
