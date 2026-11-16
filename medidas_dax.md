# Medidas DAX — Brasileirão BI

Todas as medidas estão na tabela **Partidas**.

| Medida | Fórmula | O que mostra |
|---|---|---|
| Total de Partidas | `COUNTROWS(Partidas)` | Quantidade de jogos |
| Total de Gols | `SUM(Partidas[mandante_Placar]) + SUM(Partidas[visitante_Placar])` | Soma dos gols de mandantes e visitantes |
| Media de Gols por Partida | `DIVIDE([Total de Gols], [Total de Partidas])` | Gols por jogo |
| Vitorias Mandante | `COUNTROWS(FILTER(Partidas, Partidas[vencedor] = Partidas[mandante]))` | Jogos vencidos pelo time da casa |
| Empates | `COUNTROWS(FILTER(Partidas, Partidas[vencedor] = "-"))` | Jogos empatados |
| % Vitorias Mandante | `DIVIDE([Vitorias Mandante], [Total de Partidas])` | Percentual de vitórias do mandante |
| Vitorias do Clube | `CALCULATE(COUNTROWS(Partidas), TREATAS(VALUES(Clubes[Clube]), Partidas[vencedor]))` | Vitórias de cada clube (usa TREATAS porque o relacionamento Clubes → Partidas é inativo) |
| Gols Marcados | `COUNTROWS(Gols)` | Gols registrados por atleta/clube |
| Cartoes Amarelos | `CALCULATE(COUNTROWS(Cartoes), Cartoes[cartao] = "Amarelo")` | Cartões amarelos |
| Cartoes Vermelhos | `CALCULATE(COUNTROWS(Cartoes), Cartoes[cartao] = "Vermelho")` | Cartões vermelhos |

## Tabelas criadas no Power Query

**Clubes** — lista única de clubes com o estado, a partir de mandante e visitante:

```
let
    Mand = Table.SelectColumns(Partidas, {"mandante", "mandante_Estado"}),
    Mand2 = Table.RenameColumns(Mand, {{"mandante", "Clube"}, {"mandante_Estado", "Estado"}}),
    Vis = Table.SelectColumns(Partidas, {"visitante", "visitante_Estado"}),
    Vis2 = Table.RenameColumns(Vis, {{"visitante", "Clube"}, {"visitante_Estado", "Estado"}}),
    Uniao = Table.Combine({Mand2, Vis2}),
    Distintos = Table.Distinct(Uniao, {"Clube"}),
    Ordenado = Table.Sort(Distintos, {{"Clube", Order.Ascending}})
in
    Ordenado
```

**Calendario** — um dia por linha, da primeira à última partida:

```
let
    Inicio = List.Min(Partidas[data]),
    Fim = List.Max(Partidas[data]),
    Datas = List.Dates(Inicio, Duration.Days(Fim - Inicio) + 1, #duration(1, 0, 0, 0)),
    Tabela = Table.FromList(Datas, Splitter.SplitByNothing(), {"Data"}),
    Tipo = Table.TransformColumnTypes(Tabela, {{"Data", type date}}),
    Ano = Table.AddColumn(Tipo, "Ano", each Date.Year([Data]), Int64.Type),
    Mes = Table.AddColumn(Ano, "Mes", each Date.Month([Data]), Int64.Type),
    NomeMes = Table.AddColumn(Mes, "Nome_Mes", each Date.MonthName([Data], "pt-BR"), type text),
    DiaSemana = Table.AddColumn(NomeMes, "Dia_Semana", each Date.DayOfWeekName([Data], "pt-BR"), type text)
in
    DiaSemana
```

## Relacionamentos

| De | Para | Cardinalidade | Status |
|---|---|---|---|
| Partidas[data] | Calendario[Data] | N:1 | Ativo |
| Estatisticas[partida_id] | Partidas[ID] | N:1 | Ativo |
| Gols[partida_id] | Partidas[ID] | N:1 | Ativo |
| Cartoes[partida_id] | Partidas[ID] | N:1 | Ativo |
| Estatisticas[clube] | Clubes[Clube] | N:1 | Ativo |
| Gols[clube] | Clubes[Clube] | N:1 | Ativo |
| Cartoes[clube] | Clubes[Clube] | N:1 | Ativo |
| Partidas[mandante] | Clubes[Clube] | N:1 | Inativo |
| Partidas[visitante] | Clubes[Clube] | N:1 | Inativo |

Os dois últimos ficam inativos porque o Power BI não permite dois caminhos ativos entre as mesmas tabelas.
