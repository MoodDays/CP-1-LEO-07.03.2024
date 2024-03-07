function Start-ServiceListServer {
    param (
        [string]$Url = "http://localhost:8080/services/"
    )

    # Importando o módulo
    Import-Module WebAdministration

    # Criando o objeto HttpListener
    $listener = New-Object System.Net.HttpListener

    # Adicionando a URL ao objeto HttpListener
    $listener.Prefixes.Add($Url)

    # Iniciando o listener
    $listener.Start()

    Write-Host "Servidor HTTP iniciado em $Url. Aguardando solicitações..."

    # Loop infinito para lidar com solicitações
    while ($true) {
        # Aguardando a próxima solicitação
        $context = $listener.GetContext()

        # Obtendo a resposta
        $response = $context.Response

        # Obtendo a lista de serviços ativos
        $services = Get-Service | Where-Object {$_.Status -eq "Running"} | Select-Object -ExpandProperty DisplayName

        # Convertendo a lista em formato de string
        $servicesString = $services -join "`r`n"

        # Convertendo a lista em bytes
        $buffer = [System.Text.Encoding]::UTF8.GetBytes($servicesString)

        # Definindo o tipo de conteúdo da resposta
        $response.ContentType = "text/plain"

        # Escrevendo a lista na resposta
        $response.OutputStream.Write($buffer, 0, $buffer.Length)

        # Fechando a resposta
        $response.Close()
    }

    # Parando o listener (geralmente não alcançado)
    $listener.Stop()
}

# Chamando a função para iniciar o servidor
Start-ServiceListServer
