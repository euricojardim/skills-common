# Geografia da Madeira — referência administrativa

Hierarquia administrativa da Região Autónoma da Madeira para validação do campo `locais` na extracção semântica.

## Hierarquia

```
Região Autónoma da Madeira (RAM)
├── Ilha da Madeira
│   └── 10 concelhos, 53 freguesias
└── Ilha de Porto Santo
    └── 1 concelho, 1 freguesia
```

## Concelhos e freguesias

### Ilha da Madeira

**Funchal** (capital)
- Imaculado Coração de Maria, Monte, Santa Luzia, Santa Maria Maior, São Gonçalo, São Pedro, São Roque, Sé, São Martinho, São António

**Câmara de Lobos**
- Câmara de Lobos, Curral das Freiras, Estreito de Câmara de Lobos, Jardim da Serra, Quinta Grande

**Ribeira Brava**
- Campanário, Ribeira Brava, Serra de Água, Tabua

**Ponta do Sol**
- Canhas, Madalena do Mar, Ponta do Sol

**Calheta**
- Arco da Calheta, Calheta, Estreito da Calheta, Fajã da Ovelha, Jardim do Mar, Paul do Mar, Ponta do Pargo, Prazeres

**Porto Moniz**
- Achadas da Cruz, Porto Moniz, Ribeira da Janela, Seixal

**São Vicente**
- Boaventura, Ponta Delgada, São Vicente

**Santana**
- Arco de São Jorge, Faial, Santana, São Jorge, São Roque do Faial, Ilha

**Machico**
- Água de Pena, Caniçal, Machico, Porto da Cruz, Santo António da Serra (parte)

**Santa Cruz**
- Camacha, Caniço, Gaula, Santa Cruz, Santo António da Serra (parte)

### Ilha de Porto Santo

**Porto Santo**
- Porto Santo (única freguesia)

## Topónimos frequentes em notícias

Ordenados por frequência aproximada de ocorrência editorial:

- **Funchal** — capital, sede de quase todas as instituições regionais
- **Câmara de Lobos** — segunda maior cidade
- **Machico** — peso histórico e turístico
- **Santa Cruz** — onde fica o aeroporto (Aeroporto Internacional da Madeira Cristiano Ronaldo)
- **Caniço** — densidade populacional alta
- **Caniçal** — porto comercial, zona franca
- **Curral das Freiras** — freguesia de montanha icónica
- **Porto Moniz** — piscinas naturais, turismo
- **Calheta** — desenvolvimento turístico crescente
- **Santana** — casas típicas, património

## Pontos de referência (não administrativos mas frequentes)

- **Pico Ruivo** — ponto mais alto da Madeira (1862m)
- **Pico do Areeiro** — segundo mais alto (1818m)
- **Cabo Girão** — falésia, miradouro
- **Câmara de Lobos** (baía) — pintada por Churchill
- **Marina do Funchal**
- **Madeira Tecnopolo**
- **Hospital Dr. Nélio Mendonça** — hospital central
- **Universidade da Madeira** (UMa)
- **Aeroporto Internacional da Madeira** (FNC) — Santa Cruz
- **Porto do Funchal**
- **Câmara Municipal do Funchal** (CMF)
- **Assembleia Legislativa da Região Autónoma da Madeira** (ALRAM)
- **Quinta Vigia** — sede do Governo Regional
- **ER101**, **ER104**, **VR1**, **VR2** — principais vias rodoviárias

## Vias rodoviárias principais

- **VR1** — Via Rápida (eixo Funchal-Machico)
- **VR2** — ligação ao aeroporto
- **ER101** — Estrada Regional periférica norte/oeste
- **ER104** — Estrada Regional centro
- **Via Litoral** — costa sul

## Convenções

- Sempre escrever nomes com acentuação correcta em pt-PT (Câmara, São Vicente, Caniço)
- Usar prefixo administrativo apenas quando ambíguo (ex.: "Câmara Municipal do Funchal" para distinguir de "cidade do Funchal")
- "Madeira" pode referir a ilha ou a RAM consoante contexto — o output JSON deve clarificar via `regiao`
- Para geografia fora da RAM, usar `regiao` com valor `"Continente"`, `"Açores"`, ou nome do país
