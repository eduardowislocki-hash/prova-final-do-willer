create table jogadores (
    id_jogador serial primary key,
    nickname varchar(50) not null unique,
    email varchar(100) not null unique,
    senha varchar(255) not null,
    nome_completo varchar(150) not null,
    status varchar(20) not null default 'ativo',
    data_cadastro timestamp default current_timestamp,
    constraint chk_status_jogador check (status in ('ativo', 'inativo', 'banido'))
);

create table torneios (
    id_torneio serial primary key,
    titulo varchar(100) not null,
    modalidade varchar(50) not null,
    descricao text,
    data_inicio timestamp not null,
    data_fim timestamp not null,
    status varchar(20) not null default 'agendado',
    constraint chk_status_torneio check (status in ('agendado', 'em_andamento', 'finalizado', 'cancelado'))
);

create table partidas (
    id_partida serial primary key,
    id_torneio int not null,
    id_jogador int not null,
    titulo varchar(100) not null,
    data_partida timestamp not null,
    status varchar(20) not null default 'agendado',
    data_criacao timestamp not null default current_timestamp,
    constraint fk_partida_torneio 
        foreign key (id_torneio) references torneios(id_torneio) on delete cascade,
    constraint fk_partida_jogador 
        foreign key (id_jogador) references jogadores(id_jogador) on delete restrict,
    constraint chk_status_partida 
        check (status in ('agendado', 'em_andamento', 'concluida', 'cancelada'))
);

create table historico_partidas (
    id_log serial primary key,
    id_partida int not null,
    acao varchar(20) not null,
    data_acao timestamp not null default current_timestamp,
    detalhes text,
    constraint fk_historico_partida 
        foreign key (id_partida) references partidas(id_partida) on delete cascade
);


insert into jogadores (nickname, email, senha, nome_completo, status) values
('paulomattos', 'paulo@email.com', 'senha123', 'paulo mattos', 'ativo'),
('miguelguirado', 'miguel@email.com', 'senha456', 'miguel guirado', 'ativo'),
('othelodrummond', 'othelo@email.com', 'senha789', 'othelo drummond', 'inativo');

insert into torneios (titulo, modalidade, descricao, data_inicio, data_fim, status) values
('campeonato floripa warzone', 'warzone', 'torneio regional de warzone', '2026-12-01 09:00:00', '2026-12-05 18:00:00', 'agendado'),
('liga sul de fifa', 'fifa', 'classificatória sul do campeonato de fifa', '2026-12-10 08:00:00', '2026-12-12 22:00:00', 'agendado'),
('torneio semanal warzone', 'warzone', 'competição aberta de warzone', '2026-11-30 14:00:00', '2026-11-30 22:00:00', 'agendado');

insert into partidas (id_torneio, id_jogador, titulo, data_partida, status) values
(1, 1, 'paulo vs othelo — fase de grupos', '2026-12-02 14:00:00', 'agendado'),
(2, 2, 'miguel — primeira rodada classificatória', '2026-12-10 10:00:00', 'agendado'),
(3, 1, 'paulo vs miguel — final', '2026-11-30 18:00:00', 'agendado');


create or replace view vw_painel_principal as
select
    j.nickname,
    p.titulo as confronto,
    t.modalidade,
    p.data_partida,
    p.status
from partidas p
inner join jogadores j on p.id_jogador = j.id_jogador
inner join torneios t on p.id_torneio = t.id_torneio
order by p.data_partida desc;

create or replace view vw_cronograma_partidas as
select
    p.id_partida,
    p.titulo,
    t.modalidade,
    p.data_partida,
    p.status,
    j.nickname as jogador_responsavel,
    case
        when p.data_partida > current_timestamp 
            then extract(day from p.data_partida - current_timestamp)::int
        else 0
    end as dias_restantes
from partidas p
inner join jogadores j on p.id_jogador = j.id_jogador
inner join torneios t on p.id_torneio = t.id_torneio
order by p.data_partida asc;


create or replace function fn_buscar_jogadores(termo_busca text)
returns table (
    id_jogador int,
    nickname varchar(50),
    email varchar(100),
    nome_completo varchar(150),
    status varchar(20)
) as $$
begin
    return query
    select
        j.id_jogador,
        j.nickname,
        j.email,
        j.nome_completo,
        j.status
    from jogadores j
    where
        j.nickname ilike '%' || termo_busca || '%'
        or j.email ilike '%' || termo_busca || '%'
    order by j.nickname;
end;
$$ language plpgsql;


create or replace function fn_trigger_validar_partida()
returns trigger as $$
declare
    v_status_jogador varchar(20);
begin
    select status into v_status_jogador
    from jogadores
    where id_jogador = new.id_jogador;

    if v_status_jogador is distinct from 'ativo' then
        raise exception 'não é possível vincular partida: o jogador está inativo ou banido.';
    end if;

    new.data_criacao := current_timestamp;

    insert into historico_partidas (id_partida, acao, detalhes)
    values (new.id_partida, 'criação', 'partida agendada automaticamente');

    return new;
end;
$$ language plpgsql;

create trigger trg_validar_partida
before insert on partidas
for each row
execute function fn_trigger_validar_partida();
