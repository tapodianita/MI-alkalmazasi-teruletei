# Kétnyelvű RAG: angol eredeti és magyar fordítás

Féléves feladat – *A mesterséges intelligencia alkalmazási területei* (Óbudai Egyetem, 2026/27)
Készítette: Tapodi Anita

## A feladat

Visszakereséssel bővített generálás (RAG) egy könyvsorozaton (*A hobbit* és *A Gyűrűk Ura*), az angol eredetin és a magyar fordításon mint párhuzamos korpuszon.

**Baseline:** ugyanaz a két modell visszakeresés nélkül (csak a saját tudásából válaszol).

## A lánc

```
könyvek (txt) → darabolás → beágyazás → FAISS-index → visszakeresés (top-5) → prompt → generálás
   01\_chunking      02\_indexing                          03\_evaluation
```

|elem|beállítás|
|-|-|
|darabolás|fix méret + átfedés, 200 szó / 40 szó átfedés (EN: 3592, HU: 2919 darab)|
|beágyazás|`multilingual-e5-large-instruct` (LM Studio), mindkét nyelvre ugyanaz|
|keresés|koszinusz-hasonlóság, FAISS `IndexFlatIP`, nyelvenként külön index|
|generálás|`google/gemma-4-e4b` (kicsi), `google/gemma-4-31b-qat` (nagy), LM Studio, hőmérséklet 0|
|kísérlet|15 angol–magyar kérdéspár × 2 modell × 2 nyelv × {RAG, baseline} = 120 válasz|
|mérés|Recall@1/3/5/10, MRR@10 (gold darabokkal); kézi helyességpontozás (1 / 0,5 / 0)|

## Fő eredmények

**Visszakeresés** (15 kérdés nyelvenként):

|nyelv|Recall@1|Recall@3|Recall@5|Recall@10|MRR@10|
|-|-|-|-|-|-|
|angol|0,333|0,533|0,600|0,733|0,466|
|magyar|0,267|0,400|0,467|0,533|0,358|

**Helyesség** (kézi pontozás átlaga; zárójelben az elutasított válaszok aránya):

|modell|mód|angol|magyar|
|-|-|-|-|
|nagy (31B)|baseline|0,933 (0,00)|0,600 (0,20)|
|nagy (31B)|RAG|0,633 (0,20)|0,567 (0,27)|
|kicsi (e4b)|baseline|0,167 (0,60)|0,100 (0,60)|
|kicsi (e4b)|RAG|0,433 (0,47)|0,267 (0,40)|

* A RAG a kis modellnek sokat segít (+0,27 angolul, +0,17 magyarul).
* A nagy modell a könyveket az előtanításból ismeri.
* Az angol a keresésben és a válaszokban is jobb a magyarnál.

Részletek: `results/summary.csv`, `results/retrieval\_summary.csv`, `results/en\_hu\_pairs.csv`, `results/accuracy.png`.

## A repó tartalma

```
01\_chunking.ipynb        a könyvek feldarabolása
02\_indexing.ipynb        beágyazás és vektorindex
03\_evaluation.ipynb      keresés, generálás, kiértékelés
questions.csv            a 15 kérdéspár, referenciaválaszok, gold bizonyítékrészletek
results/                 eredmények (táblázatok, ábra, kézi pontozás)
requirements.txt
```

A notebookok a lefutott állapotukban, kimenetekkel együtt szerepelnek.



## Futtatási környezet

|szerep|hardver|
|-|-|
|beágyazás|Intel Core i5-11400, 32 GB RAM, Intel UHD 730 (Vulkan)|
|nyelvi modell|AMD Ryzen 7 7800X3D, 64 GB RAM, NVIDIA RTX 5080 16 GB (CUDA)|

Mért idők:

|lépés|idő|
|-|-|
|beágyazás, angol|kb. 82 perc|
|beágyazás, magyar|kb. 89 perc|
|válasz, kis modell|átlagosan kb. 30 s|
|válasz, nagy modell|átlagosan kb. 129 s|

A válaszok 4096 tokenes keretben készültek.

