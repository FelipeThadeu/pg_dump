# pg_dump

Primeiro passo é acessar o banco que será feito o dump e executar o comando SELECT user,host FROM mysql.user;

Após isso vamos ver quais permissões os usuários vão ter SHOW GRANTS FOR ''@'%';

Vamos checar o espaço em disco da intância para ver se tem espaço suficiente para a realização do dump df -h e df -i

Após essa analise prévia vámos realizar o dump no banco, vamos criar um diretório e rodar o script dodump.sh

for DB in $(psql -h <ip/endpoint> -U <user> -W -e "\l") 
do
 pg_dump -h <ip/endpoint> -U <user> -W -e "\l" ${DB} > /tmp/dump/${DB}.dump 
done

O for vai criar uma variável chamada DB e será incoporado o resultado do \l em seguida o resultado obtido por DB será criado um arquivo para cada banco.

após finalizar, vamos transferir os arquivos gerados para a máquina que será restaurado os arquivos zip -r diretório/

será gerado um arquivo zip do diretório, que será mais rápido para a transferência, ao finalizar a compactação, vamos transferir os arquivos através o SCP

scp -i certificado.key /diretorio/aquivo.zip usuario@ip_do_host:/diretorio/

finalizando a transferencia vamos realizar a descompactação

unzip arquivo.zip

agora podemos restaurar o nosso banco com o segundo script (se for uma base que já exista realize o snapshot ou backup antes de rodar o script)

for DB in $(ls -l | awk '{print $9}' | grep dump$ | cut -d. -f 1 | grep -v psql | grep -v performance_schema | grep -v information_schema) 
do
 echo "$(date) - EXECUTANDO ${DB}"
 psql -h <ip/endpoint> -U <user> -W -e "CREATE DATABASE ${DB}"
 psql -h <ip/endpoint> -U <user> -W < ${DB}.dump 
done

O for irá varrer os arquivos excluindo as bases padrões do sistema(mysql, performance_schema, information_schema) lerá os arquivos e criará um DATABASE caso não exista, caso já exista pode ser retirada a linha mysql -u -p -h -e "CREATE DATABASE ${DB}" Na linha a seguir os arquivos dump serão importados para o DATABASE
