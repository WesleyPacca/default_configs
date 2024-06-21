Install oh-my-posh
```
winget install JanDeDobbeleer.OhMyPosh -s winget
```

Install terminal-icons
```
Install-Module Terminal-Icons -Scope CurrentUser
Install-Module DockerCompletion -Scope CurrentUser
```

edit in $PROFILE
```
# Definindo o tema
oh-my-posh init pwsh --config "$HOME\AppData\Local\oh-my-posh\themes\my_personal_theme.omp.json" | Invoke-Expression

$MaximumHistoryCount = 20000

Import-Module PSReadLine
Import-Module Get-ChildItemColor
Import-Module Terminal-Icons
Import-Module DockerCompletion

# Desabilitando impressão do ambiente virtual
$env:VIRTUAL_ENV_DISABLE_PROMPT = 1


# Importar todos os completions do diretório Scripts
$scriptsPath = "$HOME\pwsh_completions_scripts"

# Verificar se o diretório existe
if (Test-Path $scriptsPath) {
    # Obter todos os arquivos .ps1 no diretório
    $scriptFiles = Get-ChildItem -Path $scriptsPath -Filter *.ps1

    # Importar cada script encontrado
    foreach ($scriptFile in $scriptFiles) {
        . $scriptFile.FullName
    }
}

# Definindo TAB para abrir o menu de completions
Set-PSReadLineKeyHandler -Key Tab -Function MenuComplete

# Definindo o histórico de comandos
$HistoryFilePath = Join-Path ([Environment]::GetFolderPath('UserProfile')) .ps_history
Register-EngineEvent PowerShell.Exiting -Action { Get-History | Export-Clixml $HistoryFilePath } | out-null
if (Test-path $HistoryFilePath) { Import-Clixml $HistoryFilePath | Add-History }

# Definindo interações com o histórico no terminal
Set-PSReadLineOption -HistorySearchCursorMovesToEnd
Set-PSReadLineOption -PredictionSource History
Set-PSReadLineOption -ShowToolTips
Set-PSReadlineKeyHandler -Key UpArrow -Function HistorySearchBackward
Set-PSReadlineKeyHandler -Key DownArrow -Function HistorySearchForward


# Aliases / Tasks

function beautifull_tree {
    param (
        [string]$Path = (Get-Location),
        [switch]$Folders,
        [bool]$IgnoreCache = $true,
        [string[]]$IgnoreDirectories = @(),
        [switch]$Help
    )

    if ($Help) {
        Write-Output @"
beautifull_tree [-Path <string>] [-Folders] [-IgnoreCache] [-IgnoreDirectories <string[]>] [-Help]

-Path <string>:
    O caminho do diretório a ser listado. O valor padrão é o diretório atual.

-Folders:
    Lista apenas diretórios. Se este parâmetro for fornecido, os arquivos não serão listados.

-IgnoreCache:
    Ignora diretórios que contenham 'cache' no nome. O valor padrão é true.

-IgnoreDirectories <string[]>:
    Uma lista de nomes de diretórios a serem ignorados. Exemplo: @("node_modules", "dist").

-Help:
    Exibe esta mensagem de ajuda.
"@
        return
    }

    # Função recursiva para construir a estrutura do diretório
    function Get-Tree {
        param (
            [string]$SubPath,
            [int]$Level = 0
        )

        $indent = '    ' * $Level
        Get-ChildItem -Path $SubPath | ForEach-Object {
            # Verificar se o item é um diretório e se deve ser ignorado
            if ($_.PSIsContainer) {
                if ($IgnoreCache -and $_.Name -match 'cache') {
                    return
                }
                if ($IgnoreDirectories -contains $_.Name) {
                    return
                }

                Write-Output "$indent├───$($_.Name)"
                Get-Tree -SubPath $_.FullName -Level ($Level + 1)

            } elseif (-not $Folders) {
                Write-Output "$indent├───$($_.Name)"
            }
        }
    }

    # Mostrar o nome do diretório atual
    
    $currentDirName = Split-Path -Path $Path -Leaf
    Write-Output "${currentDirName}:"

    # Mostrar a estrutura do diretório
    Get-Tree -SubPath $Path
}

function fix_ps() { rm "$HOME\.ps_history" }

function ll() { Get-ChildItem | Format-Table }
function ls() { Get-ChildItem | Format-Wide }
function la() { Get-ChildItem -force }
function lb() { Get-ChildItem | Format-List }
```
Se ao abrir o terminal, você obter a mensagem abaixo, execute ```fix_ps```

