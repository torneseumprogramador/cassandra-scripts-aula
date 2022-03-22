# install cassandra MacOs
https://medium.com/@manishyadavv/how-to-install-cassandra-on-mac-os-d9338fcfcba4

# treinamento bacana
https://www.youtube.com/watch?v=Ji6I-T0sz4o

# install
brew install cassandra

# start cassandra
cassandra -f

# acessar no terminal CLI
cqlsh

# acessar com usuário e senha
cqlsh 127.0.0.1 -u cassandra -p cassandra

# documentação cassandra
https://docs.datastax.com/en/cql-oss/3.3/cql/cql_using/useAboutCQL.html

# create database(keyspace) cassandra
describe keyspaces; 
### outra forma de ver os keyspaces
SELECT * from system_schema.keyspaces;

https://www.geeksforgeeks.org/create-database-in-cassandra/

### partition key = primary key
- partição criada organizando itens clusterizados 
- para criar mais de uma partição em uma tabela utilizar o esquema "primary key((cpf,telefone), nome)" neste caso, cpf e telefone é partition key combinados e nome é cluester key
### clustering key = itens na criação da tabela para Ordenação 

# A ordenação de uma tabela é sempre feita através de seus partition e cluster keys definidos na criação da tabela
https://youtu.be/S9rmf4X7E_E


# new cassandra create keyspace
https://docs.datastax.com/en/cql-oss/3.3/cql/cql_using/useExampleCreatingKeyspace.html

# scrit create 
CREATE KEYSPACE IF NOT EXISTS App_data_danilo WITH REPLICATION = { 'class' : 'NetworkTopologyStrategy', 'datacenter1' : 3 };

# verificar databases criadas
describe keyspaces; 

# describe uma database
describe App_data_danilo; 

# usar o database
USE App_data_danilo;

# data type cassandra
https://cassandra.apache.org/doc/latest/cassandra/cql/types.html

# show tables, mostrar tabelas
describe tables;

# create table
https://docs.datastax.com/en/cql-oss/3.3/cql/cql_using/useCreateTable.html
CREATE TABLE alunos ( id UUID PRIMARY KEY, nome text, data_nasc timestamp, nacionalidade text, peso float, altura float, idade int);

# insert data, inserindo registro com a mesmo id de partição ele não inclui, ele atualiza
INSERT INTO alunos (id, nome, nacionalidade, data_nasc, peso, altura, idade)
VALUES (5b6962dd-3f90-4c93-8f61-eabfa4a803e2, 'Danilo','Brasileiro', '1983-09-12 09:00:00', 75.4, 1.75, 28);

insert into clientes(id, altura, data_nasc, nacionalidade, nome, peso)values(5b6962dd-3f90-4c93-8f61-eabfa4a803e2, 'alto', '1983-12-09', 'Brasileira', 'Danilo', '110 kilos') ;

# select filter erro
SELECT * FROM alunos where nome = 'Danilo';

# select filter, usar em ultimos casos, pois isso perde a performance do banco, faz com que ele busque forçado em todos os itens do cluster
SELECT * FROM alunos where nome = 'Danilo' ALLOW FILTERING;

# descrever tabela
desc clientes;

# insert data
INSERT INTO clientes (id, nome, nacionalidade) VALUES (5b6962dd-3f90-4c93-8f61-eabfa4a803e2, 'Danilo','Brasileiro');
INSERT INTO calendar (race_id, race_start_date, race_end_date, race_name) VALUES 
  (201, '2015-02-18', '2015-02-22', $$Women's Tour of New Zealand$$);
  
  
# insert dados temporários TTL (Time to live)
INSERT INTO clientes (id, nome, nacionalidade) VALUES (5b6962dd-3f90-4c93-8f61-eabfa4a80355, 'Joao','Brasileiro') using ttl 10; # 10 segundos
select * from clientes; # desaparece o registro em 10 segundos

# exercicio criar atualizar apagar
# criar tabela
CREATE TABLE cyclist_expenses ( 
  cyclist_name text, 
  balance float STATIC, 
  expense_id int, 
  amount float, 
  description text, 
  paid boolean, 
  PRIMARY KEY (cyclist_name, expense_id) 
);

# insert
BEGIN BATCH
  INSERT INTO cyclist_expenses (cyclist_name, balance) VALUES ('Vera ADRIAN', 0) IF NOT EXISTS;
  INSERT INTO cyclist_expenses (cyclist_name, expense_id, amount, description, paid) VALUES ('Vera ADRIAN', 1, 7.95, 'Breakfast', false);
  APPLY BATCH;

# update
UPDATE cyclist_expenses SET balance = -7.95 WHERE cyclist_name = 'Vera ADRIAN' IF balance = 0;

# update com transaction
BEGIN BATCH
INSERT INTO cyclist_expenses (cyclist_name, expense_id, amount, description, paid) VALUES ('Vera ADRIAN', 2, 13.44, 'Lunch', true);
INSERT INTO cyclist_expenses (cyclist_name, expense_id, amount, description, paid) VALUES ('Vera ADRIAN', 3, 25.00, 'Dinner', false);
UPDATE cyclist_expenses SET balance = -32.95 WHERE cyclist_name = 'Vera ADRIAN' IF balance = -7.95;
APPLY BATCH;

# + update
BEGIN BATCH
UPDATE cyclist_expenses SET balance = 0 WHERE cyclist_name = 'Vera ADRIAN' IF balance = -32.95;
UPDATE cyclist_expenses SET paid = true WHERE cyclist_name = 'Vera ADRIAN' AND expense_id = 1 IF paid = false;
UPDATE cyclist_expenses SET paid = true WHERE cyclist_name = 'Vera ADRIAN' AND expense_id = 3 IF paid = false;
APPLY BATCH;

# + insert
BEGIN LOGGED BATCH
INSERT INTO cyclist_names (cyclist_name, race_id) VALUES ('Vera ADRIAN', 100);
INSERT INTO cyclist_by_id (race_id, cyclist_name) VALUES (100, 'Vera ADRIAN');
APPLY BATCH;


# exercicio de select
# tabela
CREATE TABLE rank_by_year_and_name ( 
  race_year int, 
  race_name text, 
  cyclist_name text, 
  rank int, 
  PRIMARY KEY ((race_year, race_name), rank) );

# insert
INSERT INTO rank_by_year_and_name (race_year, race_name, cyclist_name, rank) 
   VALUES (2015, 'Tour of Japan - Stage 4 - Minami > Shinshu', 'Benjamin PRADES', 1);
INSERT INTO rank_by_year_and_name (race_year, race_name, cyclist_name, rank) 
   VALUES (2015, 'Tour of Japan - Stage 4 - Minami > Shinshu', 'Adam PHELAN', 2);
INSERT INTO rank_by_year_and_name (race_year, race_name, cyclist_name, rank) 
   VALUES (2015, 'Tour of Japan - Stage 4 - Minami > Shinshu', 'Thomas LEBAS', 3);
INSERT INTO rank_by_year_and_name (race_year, race_name, cyclist_name, rank) 
   VALUES (2015, 'Giro d''Italia - Stage 11 - Forli > Imola', 'Ilnur ZAKARIN', 1);
INSERT INTO rank_by_year_and_name (race_year, race_name, cyclist_name, rank) 
   VALUES (2015, 'Giro d''Italia - Stage 11 - Forli > Imola', 'Carlos BETANCUR', 2);
INSERT INTO rank_by_year_and_name (race_year, race_name, cyclist_name, rank) 
   VALUES (2014, '4th Tour of Beijing', 'Phillippe GILBERT', 1);
INSERT INTO rank_by_year_and_name (race_year, race_name, cyclist_name, rank)  
   VALUES (2014, '4th Tour of Beijing', 'Daniel MARTIN', 2);
INSERT INTO rank_by_year_and_name (race_year, race_name, cyclist_name, rank)  
   VALUES (2014, '4th Tour of Beijing', 'Johan Esteban CHAVES', 3);

# select 
SELECT * FROM rank_by_year_and_name;


# create order by desc ou asc cassandra
https://www.geeksforgeeks.org/arranging-clustering-column-in-descending-order-in-cassandra/
# sem ordenar
CREATE TABLE Emp_track (
  emp_no int,
  dept text,
  name text,
  PRIMARY KEY (dept, emp_no)
); 
insert into Emp_track(emp_no, dept, name) values (101, 'database', 'Ashish'); 
insert into Emp_track(emp_no, dept, name) values (102, 'database', 'rana'); 
insert into Emp_track(emp_no, dept, name) values (103, 'database', 'zishan'); 
insert into Emp_track(emp_no, dept, name) values (104, 'database', 'abi'); 
insert into Emp_track(emp_no, dept, name) values (105, 'database', 'kartikey');
select * 
from Emp_track; 

# ordenando decrescente
CREATE TABLE Emp_track (
  emp_no int,
  dept text,
  name text,
  PRIMARY KEY (dept, emp_no)
)
WITH CLUSTERING ORDER BY (emp_no asc); 

insert into Emp_track(emp_no, dept, name) values (101, 'database', 'Ashish'); 
insert into Emp_track(emp_no, dept, name) values (102, 'database', 'rana'); 
insert into Emp_track(emp_no, dept, name) values (103, 'database', 'zishan'); 
insert into Emp_track(emp_no, dept, name) values (104, 'database', 'abi'); 
insert into Emp_track(emp_no, dept, name) values (105, 'database', 'kartikey'); 

select * 
from Emp_track;

# select com filtro
SELECT * FROM rank_by_year_and_name WHERE cyclist_name = 'Phillippe GILBERT';
# create select com filtro
CREATE TABLE cyclist_cat_pts ( category text, points int, id UUID,lastname text, PRIMARY KEY (category, points) ); 
INSERT INTO cyclist_cat_pts (category, points, id, lastname) 
   VALUES ('Primaria', 1, 5b6962dd-3f90-4c93-8f61-eabfa4a803e2, 'Santos');
SELECT * FROM cyclist_cat_pts WHERE category = 'GC' ORDER BY points ASC;


# select com campo especídico
SELECT race_name FROM rank_by_year_and_name;

# select com limit
SELECT race_name FROM rank_by_year_and_name limit 3;

# create select com filtro
CREATE TABLE cyclist_cat_pts ( category text, points int, id UUID,lastname text, PRIMARY KEY (category, points) ); 
INSERT INTO cyclist_cat_pts (category, points, id, lastname) 
   VALUES ('Primaria', 1, 5b6962dd-3f90-4c93-8f61-eabfa4a803e2, 'Santos');
SELECT * FROM cyclist_cat_pts WHERE category = 'GC' ORDER BY points ASC;

# select renomeando coluna
SELECT race_name, point_id, lat_long AS CITY_LATITUDE_LONGITUDE FROM cycling.route;

# alterando coluna da tabela
ALTER TABLE cycling.cyclist_alt_stats ADD age int;

ALTER TABLE cycling.cyclist_alt_stats ADD favorite_color varchar;
ALTER TABLE cycling.cyclist_alt_stats ALTER favorite_color TYPE text;

# qyerys
CREATE TABLE rank_by_year_and_name ( 
  race_year int, 
  race_name text, 
  cyclist_name text, 
  rank int, 
  PRIMARY KEY ((race_year, race_name), rank) 
);

### querys permitidas
SELECT * FROM rank_by_year_and_name WHERE race_year=2015 AND race_name='Tour of Japan - Stage 4 - Minami > Shinshu';

### querys permitidas, sempre seguir a ordem em que foi criado as partitions keys
select * from rank_by_year_and_name where race_year=2014 and race_name='4th Tour of Beijing' and rank=1 and cyclist_name='Phillippe GILBERT'  ALLOW FILTERING;
### querys permitidas, mudando orgem, maos obedecendo a partition key primeiro
select * from rank_by_year_and_name where race_name='4th Tour of Beijing' and race_year=2014 and rank=1

### querys não permitidas, item, cyclist_name não é partition key e não é cluster order key, portanto não é permitido
select * from rank_by_year_and_name where race_name='4th Tour of Beijing' and race_year=2014 and rank=1 and cyclist_name='Phillippe GILBERT';
### querys não permitidas, não é permitido buscar dados sem colocar as duas partições definidas na tabela
select * from rank_by_year_and_name where rank=1 and race_year=2014;


SELECT * FROM cycling.rank_by_year_and_name WHERE race_year=2015;

# criando index, outra forma que permite busca entre as colunas sem respeirar a ordem dos primary keys, porem baixa a performance, use com cuidado
CREATE INDEX ryear ON cycling.rank_by_year_and_name (race_year);
SELECT * FROM cycling.rank_by_year_and_name WHERE race_year=2015;

## exemplo de indice criado que permite uma busca sem a ordem da partition key
CREATE INDEX clientes_altura ON clientes(altura);
SELECT * FROM clientes WHERE altura='alto';


# select like

CREATE TABLE cycling.cyclist_name ( 
  id UUID PRIMARY KEY, 
  lastname text, 
  firstname text
);


CREATE CUSTOM INDEX  fn_prefix ON cyclist_name (firstname)
USING 'org.apache.cassandra.index.sasi.SASIIndex';


SELECT * FROM cyclist_name WHERE firstname LIKE 'M%';
SELECT * FROM cyclist_name WHERE firstname LIKE 'Mic%';

# drop table, apagar tabela, remover tabela
DROP TABLE rank_by_year_and_name;


# caso onde a ordem vai de acordo com as partições, olhar o select no campo rank

CREATE TABLE rank_by_year_and_name ( 
  race_year int, 
  race_name text, 
  cyclist_name text, 
  rank int, 
  PRIMARY KEY ((race_year), rank, race_name)
)
WITH CLUSTERING ORDER BY (rank asc); 


INSERT INTO rank_by_year_and_name (race_year, race_name, cyclist_name, rank) 
   VALUES (2015, 'Tour of Japan - Stage 4 - Minami > Shinshu', 'Benjamin PRADES', 1);
INSERT INTO rank_by_year_and_name (race_year, race_name, cyclist_name, rank) 
   VALUES (2015, 'Tour of Japan - Stage 4 - Minami > Shinshu', 'Adam PHELAN', 2);
INSERT INTO rank_by_year_and_name (race_year, race_name, cyclist_name, rank) 
   VALUES (2015, 'Tour of Japan - Stage 4 - Minami > Shinshu', 'Thomas LEBAS', 3);
INSERT INTO rank_by_year_and_name (race_year, race_name, cyclist_name, rank) 
   VALUES (2015, 'Giro d''Italia - Stage 11 - Forli > Imola', 'Ilnur ZAKARIN', 1);
INSERT INTO rank_by_year_and_name (race_year, race_name, cyclist_name, rank) 
   VALUES (2015, 'Giro d''Italia - Stage 11 - Forli > Imola', 'Carlos BETANCUR', 2);
INSERT INTO rank_by_year_and_name (race_year, race_name, cyclist_name, rank) 
   VALUES (2014, '4th Tour of Beijing', 'Phillippe GILBERT', 1);
INSERT INTO rank_by_year_and_name (race_year, race_name, cyclist_name, rank)  
   VALUES (2014, '4th Tour of Beijing', 'Daniel MARTIN', 2);
INSERT INTO rank_by_year_and_name (race_year, race_name, cyclist_name, rank)  
   VALUES (2014, '4th Tour of Beijing', 'Johan Esteban CHAVES', 3);


### caso onde a ordem vai de acordo com as partições
select * from rank_by_year_and_name;

### também e possivel ordenar os dados dentro das partiçoes como decrescnte e crescente
select * from rank_by_year_and_name where race_year = 2014 order by rank desc;
select * from rank_by_year_and_name where race_year = 2014 order by rank asc;

# casos que insert também faz update
### os dois inserts irão deixar somente uma linha
INSERT INTO rank_by_year_and_name (race_year, race_name, cyclist_name, rank)  VALUES (2014, '4th Tour of Beijing', 'Johan Esteban CHAVES', 3);
INSERT INTO rank_by_year_and_name (race_year, race_name, cyclist_name, rank)  VALUES (2014, '4th Tour of Beijing', 'Johan Esteban CHAVES', 3);
select * from rank_by_year_and_name where race_year = 2014;
### este insert vai fazer update da coluna cyclist_name
INSERT INTO rank_by_year_and_name (race_year, race_name, cyclist_name, rank)  VALUES (2014, '4th Tour of Beijing', 'Johan Esteban CHAVES 1', 3);
select * from rank_by_year_and_name where race_year = 2014;
### este realmente insere uma linha, pois o partition key e cluster key combinam de forma diferente
INSERT INTO rank_by_year_and_name (race_year, race_name, cyclist_name, rank)  VALUES (2014, '4th Tour of Beijing', 'Johan Esteban CHAVES 4', 4);
select * from rank_by_year_and_name where race_year = 2014;

# apagar dados
### este permite apagar pois estao dentro da mesma partição
delete from rank_by_year_and_name where race_year = 2014; 

### este não permite apagar pois, precisa filtrar pela partição
delete from rank_by_year_and_name;

