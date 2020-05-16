# azcopy
Copiar FileShare para BloStorage com envio de e-mail.

#Ler a senha e armazenar seguramente em um arquivo de texto. Essa linha não deve permancer no script.
read-host "Digite a senha" -assecurestring | convertfrom-securestring | out-file C:\Bat\pass.txt

#Armazena a senha existente no txt em variavel.
$password = Get-Content C:\Bat\pass.txt | ConvertTo-SecureString

#Informa a conta de e-mail de envio.
$Credential = New-Object -TypeName System.Management.Automation.PSCredential -ArgumentList alan.santos@bhs.com.br,$password

#Pasta de trabalho.
$FolderPath = " C:\Bat"

#Coleta no momento da execução.
$Date = Get-Date -Format "dd-MM-yyyy"

#Parametros para cópia, utilizando SAS, de um fileshare para blobstorage.
$URLSOURCE = "https://<source-storage-account-name>.file.core.windows.net/<container-name>/<pasta_data_copia><SAS-token>"

$URLDESTINY = "https://<source-storage-account-name>.blob.core.windows.net/<container-name>/<pasta_data_copia><SAS-token>"

#Alteração da URL de destino, troca "pasta_data_copia" pela variavel $Date.
$NEWURL = $URLDESTINY.Replace("pasta_data_copia",$Date)

#Arquivo que receberá os dados do AzCopy.
$FilePath = "$FolderPath\AzureArchive-$Date.txt"

#Exportação do resultado para o arquivo.
$Result = azcopy copy $URLSOURCE $NEWURL --recursive | Out-file $FilePath

#Consulta dos dados em texto puro.
$Content = Get-Content $FilePath -Raw

#Condicional para envio de e-mail.
If ($Content -match "Job Status: Completed")
{
    Send-MailMessage -From 'Exemplo <exemplo@email.com>' -To 'Exemplo <exemplo@email.com> -Subject "Copia realizada com sucesso!" -Body "Log AzCopy Copia de FileShare para BlobStorage em anexo." -Attachments $FilePath -SmtpServer 'smtp.outlook.com' -Credential $Credential  -UseSsl
}
Else {
    Send-MailMessage -From 'Exemplo <exemplo@email.com> -To 'Exemplo <exemplo@email.com> -Subject "Copia falhou - Entrar em contato com o Administrador" -Body "Log AzCopy Copia de FileShare para BlobStorage em anexo." -Attachments $FilePath -SmtpServer 'smtp.outlook.com' -Credential $Credential  -UseSsl
}

#Exclui arquivo ao final do processo.
Remove-Item $FilePath
