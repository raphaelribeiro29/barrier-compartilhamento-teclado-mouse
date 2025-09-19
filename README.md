# barrier-compartilhamento-teclado-mouse
Repositório com a configuração para compartilhamento de teclado e mouse entre duas máquinas na mesma rede.

## Problema
Utilizar o mesmo teclado e mouse em 2 (duas) máquinas diferentes conectados na mesma rede (Sistemas Operacionais: Windows 10 Pro - Versão 22H2 e Ubuntu 24.04.3 LTS), sendo que o teclado e mouse estão conectado apenas em 1 (um) computador (SO: Windows 10 Pro).

## Ferramentas utilizadas
Barrier (https://sourceforge.net/projects/barrier.mirror/): O Barrier é um software que funciona como um switch KVM (teclado, vídeo e mouse), permitindo que você use um único mouse e teclado para controlar vários computadores diferentes.

## Instalação no Windows (que será chamada de máquina Server)
No link anterior é possivel baixar o instalador executável para Windows. A versão utilizada foi baixada através do link: `https://sourceforge.net/projects/barrier.mirror/files/v2.4.0/BarrierSetup-2.4.0-release.exe/download`.

### Configuração na máquina Server
Após a instalação, o programa deve ser executado, deve ser selecionada a opção "Server", após deve clicar no botão "Configurar servidor", será exibida uma nova janela que deverá clicar no ícone de monitor no canto superior da janela e arrastar para a parte inferior colocando no local onde estão posicionado os monitores dos computadores utilizados. É possivel alterar o nome do monitor para que facilite a identificação das máquinas. Após configurar o monitor, deve ser desabilitado a utilização do SSL, para isso, deve clicar no Menu "Barrier", em seguida no item "Change Settings", deve desabilitar a opção "Enable SSL" e clicar no botao "OK". Após a configuração, pode fechar a janela, clicar em "Aplicar" e depois em "Iniciar".

### Erro apresentado na máquina Server
Após a instalação e execução do programa, foi apresentado o erro de Certificado inexistente, conforme log apresentado pelo programa.
```
INFO: OpenSSL 1.0.2l  25 May 2017
ERROR: ssl certificate doesn't exist: C:\Users\<nome_de_usuario>\AppData\Local\Barrier\SSL\Barrier.pem
```

A solução para resolver o problema foi encontrada na Issue #1377 através do link (https://github.com/debauchee/barrier/issues/1377).
Deve ser executado o PowerShell e executar os comandos abaixo para que seja possivel gerar o Certificado utilizado na ferramenta.
```
$cert = New-SelfSignedCertificate -DnsName Barrier -KeyExportPolicy Exportable

# Public key to Base64
$CertBase64 = [System.Convert]::ToBase64String($cert.RawData, 'InsertLineBreaks')

# Private key to Base64
$RSACng = [System.Security.Cryptography.X509Certificates.RSACertificateExtensions]::GetRSAPrivateKey($cert)
$KeyBytes = $RSACng.Key.Export([System.Security.Cryptography.CngKeyBlobFormat]::Pkcs8PrivateBlob)
$KeyBase64 = [System.Convert]::ToBase64String($KeyBytes, [System.Base64FormattingOptions]::InsertLineBreaks)

# Put it all together
$Pem = @"
-----BEGIN PRIVATE KEY-----
$KeyBase64
-----END PRIVATE KEY-----
-----BEGIN CERTIFICATE-----
$CertBase64
-----END CERTIFICATE-----
"@

# Output to file
$Pem | Out-File -FilePath $env:LOCALAPPDATA\Barrier\SSL\Barrier.pem -Encoding Ascii
```

## Instalação no Ubuntu (que será chamada de máquina Client)
A ferramenta pode ser instalada diretamente pela Central de Aplicações, pesquisar pelo programa `Barrier` e deve ser selecionado o de ícone: ![Ícone Barrier](./recursos/icone.png)

### Configuração na máquina Client
Após a instalação, o programa deve ser executado, deve ser selecionada a opção "Client", caso a opção "Auto config" esteja habilitada mas não funcione, deve ser desabilitada para que seja possivel preencher o endereço IP da máquina Servidor. Após informar o endereço IP, deve clicar no botão "Aplicar" e em seguida no botão "Iniciar".

---

## Arquivo de Recursos: Tutorial Barrier - Autor: Giuseppe C. Cervo
Para facilitar, foi localizado um arquivo PDF com o tutorial para instalação e configuração da ferramente Barrier. O arquivo encontra-se na pasta "recursos" deste repositório, tendo sido encontrado na internet através do link: https://www.ufsm.br/app/uploads/sites/762/2020/11/Tutorial-Barrier.pdf.
