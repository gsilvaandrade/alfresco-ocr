# alfresco-ocr
Add OCR module on Alfresco Community

Coipe repo JAR para /opt/alfresco/modules/platform  (Caso não exista, basta criá-lo).

Coipe share JAR para /opt/alfresco/modules/share    (Caso não exista, basta criá-lo).

# Atualização do tesseract para 4.0
sudo add-apt-repository ppa:alex-p/tesseract-ocr

sudo apt-get update

sudo apt-get upgrade

# Intalação dos dados para português
apt-get install tesseract-ocr-por

# Instalação do pypdfocr e dependências
apt install python-pip

pip install pypdfocr

pip install pyyaml

# O pypdfocr só funciona com va versão 3.4.0 do reportlab, então precisa instalar a versão correta
# Referência: https://github.com/virantha/pypdfocr/issues/80#issuecomment-435129520

pip uninstall reportlab
pip install reportlab==3.4.0

#Como o pypdfocr espera apenas um parâmetro de entrada, e não de saída, o script abaixo foi criado para eliminar o último parâmetro que o alfresco envia, que seria o documento de saída.
# Referência: https://stackoverflow.com/a/20398578/1571186

# Criar script ocr.sh contendo:

#!/usr/bin/env bash
# set -o xtrace # Uncomment for debugging/troubleshooting

array=( "$@" )
unset "array[${#array[@]}-1]"
/usr/local/bin/pypdfocr "${array[@]}"

# Permissão de execução para o script

chmod +x ocr.sh

# Instalação de dependências

apt install gcc libjpeg-dev minizip zlib1g-dev python-dev

# Configuração no Alfresco
vi ./tomcat/shared/classes/alfresco-global.properties

# alfresco-global.properties
# PYPDFOCR
ocr.command=/opt/alfresco/scripts/ocr.sh
ocr.output.verbose=true
ocr.output.file.prefix.command=

ocr.extra.commands=-v -l por
ocr.server.os=linux


pip uninstall reportlab
pip install reportlab==3.4.0

# O pypdfocr tem alguns problemas que precisam ser corrigidos, então fazer as edições seguintes no arquivo /usr/local/lib/python2.7/dist-packages/pypdfocr/pypdfocr_tesseract.py

# Aplicar os ajustes

# Ignorar verificação da versão fora do padrão
# Linha 98, comentar e incluir a próxima no lugar
#ver = [int(x) for x in ver_str.split('.')]
ver = [4]

# Corrigir erro: Error, unknown command line argument '-psm'
# Linha 164, incluído o segundo -, como --psm
cmd = '%s "%s" "%s" --psm 1 -c hocr_font_info=1 -l %s hocr' % (self.binary, img_filename, basename, self.lang)
