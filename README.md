# comandos-SQLite3

É sempre bom ter tudo que você precisa de forma rápida e simples.

Criando uma tabela

CRUD

Backup

Relacionando tabelas

Escrevi este post para um mini tutorial de SQLite3. Através do terminal:
Criando uma tabela

    Criando um banco de dados.

$ sqlite3 Clientes.db

    A Ajuda.

sqlite> .help

    Criando a tabela clientes.

sqlite> CREATE TABLE clientes(
   ...> id INTEGER NOT NULL PRIMARY KEY AUTOINCREMENT,
   ...> Nome VARCHAR(100) NOT NULL,
   ...> CPF VARCHAR(11) NOT NULL,
   ...> Email VARCHAR(20) NOT NULL,
   ...> Fone VARCHAR(20),
   ...> UF VARCHAR(2) NOT NULL
   ...> );

Nota: Se usamos AUTOINCREMENT não precisamos do NOT NULL.

sqlite> CREATE TABLE clientes(
   ...> id INTEGER PRIMARY KEY AUTOINCREMENT,
   ...> ...

    Visualizando o código SQL que criou a tabela.

sqlite> .schema clientes

    Visualizando todas as tabelas existentes.

sqlite> .table

    Saindo do SQLite3.

sqlite> .exit

CRUD

Abra um editor de texto e salve um arquivo com o nome inserirdados.sql.

$ gedit inserirdados.sql

E digite a inserção de alguns dados.

INSERT INTO clientes VALUES(1, 'Regis', '00000000000', 'rg@email.com', '1100000000', 'SP');
INSERT INTO clientes VALUES(2, 'Abigail', '11111111111', 'abigail@email.com', '1112345678', 'RJ');
INSERT INTO clientes VALUES(3, 'Benedito', '22222222222', 'benedito@email.com', '1187654321', 'SP');
INSERT INTO clientes VALUES(4, 'Zacarias', '33333333333', 'zacarias@email.com', '1199999999', 'RJ');

Nota: No caso do INSERT INTO não precisamos numerar, basta trocar o número do id por NULL, exemplo:

INSERT INTO clientes VALUES(NULL, 'Carlos', '99999999999', 'carlos@email.com', '118888-8888', 'SP');

    Importe estes comandos no sqlite.

$ sqlite3 Clientes.db < inserirdados.sql

    Abra o SQLite3 novamente, e visualize os dados.

$ sqlite3 Clientes.db
sqlite> SELECT * FROM clientes;

    Você pode exibir o nome das colunas digitando

sqlite> .header on

    Para escrever o resultado num arquivo externo digite

sqlite> .output resultado.txt
sqlite> SELECT * FROM clientes;
sqlite> .exit
$ cat resultado.txt

    Adicionando uma nova coluna na tabela clientes.

sqlite> ALTER TABLE clientes ADD COLUMN bloqueado BOOLEAN;

No SQLite3 os valores para boolean são 0 (falso) e 1 (verdadeiro).

    Visualizando as colunas da tabela clientes.

sqlite> PRAGMA table_info(clientes);

    Alterando os valores do campo bloqueado.

sqlite> UPDATE clientes SET bloqueado=0; -- comentario: Atualiza todos os registros para Falso.
sqlite> UPDATE clientes SET bloqueado=1 WHERE id=1; -- Atualiza apenas o registro com id=1 para Verdadeiro.
sqlite> UPDATE clientes SET bloqueado=1 WHERE UF='RJ'; -- Atualiza para Verdadeiro todos os registros com UF='RJ'.

Faça um SELECT novamente para ver o resultado.

    Deletando registros.

sqlite> DELETE FROM clientes WHERE id=4;

Cuidado: se você não usar o WHERE e escolher um id você pode deletar todos os registros da tabela.

    Você pode exibir os dados na forma de coluna.

sqlite> .mode column

Backup

$ sqlite3 Clientes.db .dump > clientes.sql
$ cat clientes.sql
PRAGMA foreign_keys=OFF;
BEGIN TRANSACTION;
CREATE TABLE clientes(
id INTEGER NOT NULL PRIMARY KEY AUTOINCREMENT,
Nome VARCHAR(100) NOT NULL,
CPF VARCHAR(11) NOT NULL,
Email VARCHAR(20) NOT NULL,
Fone VARCHAR(20),
UF VARCHAR(2) NOT NULL
);
INSERT INTO "clientes" VALUES(1,'Regis','00000000000','rg@email.com','1100000000','SP');
INSERT INTO "clientes" VALUES(2,'Abigail','11111111111','abigail@email.com','1112345678','RJ');
INSERT INTO "clientes" VALUES(3,'Benedito','22222222222','benedito@email.com','1187654321','SP');
INSERT INTO "clientes" VALUES(4,'Zacarias','33333333333','zacarias@email.com','1199999999','RJ');
COMMIT;

Pronto, se corromper o seu banco de dados, você pode recuperá-lo:

$ mv Clientes.db Clientes.db.old
$ sqlite3 Clientes_recuperado.db < clientes.sql
$ sqlite3 Clientes_recuperado.db 'SELECT * FROM clientes;'

Faça um SELECT novamente para ver o resultado do novo banco de dados.
Relacionando tabelas

Todos devem saber que num banco de dados relacional a chave estrangeira ou FOREIGN KEY tem um papel importante no relacionamento entre duas tabelas. Veremos aqui como relacionar duas tabelas.

Primeiros façamos um backup do nosso bd.

$ sqlite3 Clientes.db .dump > clientes.sql

Apenas para relembrar, vamos ver qual é a nossa tabela...

$ sqlite3 Clientes.db
sqlite> .tables
clientes
sqlite> .header on
sqlite> .mode column

E quais são seus registros.

sqlite> SELECT * FROM clientes;

id          Nome        CPF          Email         Fone        UF          bloqueado
----------  ----------  -----------  ------------  ----------  ----------  ----------
1           Regis       00000000000  rg@email.com  1100000000  SP          1
2           Abigail     11111111111  abigail@emai  1112345678  RJ          1
3           Benedito    22222222222  benedito@ema  1187654321  SP          0

Então vamos criar duas novas tabelas: cidades e clientes_novo.

sqlite> CREATE TABLE cidades(
   ...> id INTEGER PRIMARY KEY AUTOINCREMENT,
   ...> cidade TEXT,
   ...> uf VARCHAR(2)
   ...> );
   ...> CREATE TABLE clientes_novo(
   ...> id INTEGER PRIMARY KEY AUTOINCREMENT,
   ...> Nome VARCHAR(100) NOT NULL,
   ...> CPF VARCHAR(11) NOT NULL,
   ...> Email VARCHAR(20) NOT NULL,
   ...> Fone VARCHAR(20),
   ...> bloqueado BOOLEAN,
   ...> cidade_id INTEGER,
   ...> FOREIGN KEY (cidade_id) REFERENCES cidades(id)
   ...> );

Segundo Sqlite Drop Column, não tem como "deletar" uma coluna, então precisamos criar uma nova tabela clientes_novo com os campos que precisamos e copiar os dados da primeira tabela para esta.

sqlite> INSERT INTO clientes_novo (id, Nome, CPF, Email, Fone, bloqueado)
   ...> SELECT id, Nome, CPF, Email, Fone, bloqueado FROM clientes;

Veja que selecionamos os campos da tabela clientes e a inserimos em clientes_novo. Note que não copiamos o campo UF porque agora ele é da tabela cidades.

Agora podemos deletar a tabela "antiga".

sqlite> DROP TABLE clientes;

E renomear a nova tabela.

sqlite> ALTER TABLE clientes_novo RENAME TO clientes;

Veja o resultado da nova tabela.

sqlite> SELECT * FROM clientes;

id          Nome        CPF          Email         Fone        bloqueado   cidade_id
----------  ----------  -----------  ------------  ----------  ----------  ----------
1           Regis       00000000000  rg@email.com  1100000000  1
2           Abigail     11111111111  abigail@emai  1112345678  1
3           Benedito    22222222222  benedito@ema  1187654321  0

Agora você terá que popular as cidades e definir a cidade_id em cada cliente. Lembrando que a chave é AUTOINCREMENT, então use NULL.

sqlite> INSERT INTO cidades VALUES (NULL,'Campinas','SP');
sqlite> INSERT INTO cidades VALUES (NULL,'Sao Paulo','SP');
sqlite> INSERT INTO cidades VALUES (NULL,'Rio de Janeiro','RJ');

Veja os registros da tabela cidades.

sqlite> SELECT * FROM cidades;

id          cidade      uf
----------  ----------  ----------
1           Campinas    SP
2           Sao Paulo   SP
3           Rio de Jan  RJ

Agora precisamos atualizar a cidade_id de cada cliente.

sqlite> UPDATE clientes SET cidade_id = 3 WHERE id = 1;
sqlite> UPDATE clientes SET cidade_id = 1 WHERE id = 2;
sqlite> UPDATE clientes SET cidade_id = 2 WHERE id = 3;

Resultado.

sqlite> SELECT * FROM clientes;

id          Nome        CPF          Email         Fone        bloqueado   cidade_id
----------  ----------  -----------  ------------  ----------  ----------  ----------
1           Regis       00000000000  rg@email.com  1100000000  1           3
2           Abigail     11111111111  abigail@emai  1112345678  1           1
3           Benedito    22222222222  benedito@ema  1187654321  0           2

Façamos um INNER JOIN para visualizar todos os dados, inclusive a cidade e o uf.

sqlite> SELECT * FROM clientes INNER JOIN cidades ON clientes.cidade_id = cidades.id;

id          Nome        CPF          Email         Fone        bloqueado   cidade_id   cidade          uf
----------  ----------  -----------  ------------  ----------  ----------  ----------  --------------  --
1           Regis       00000000000  rg@email.com  1100000000  1           3           Rio de Janeiro  RJ
2           Abigail     11111111111  abigail@emai  1112345678  1           1           Campinas        SP
3           Benedito    22222222222  benedito@ema  1187654321  0           2           Sao Paulo       SP


Referências

SQLite.org

Introduction to Foreign Key Constraints

Making Other Kinds Of Table Schema Changes

Sqlite Drop Column

