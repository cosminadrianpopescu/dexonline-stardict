Introducere
========================================

Acesta este un dictionar roman, bazat pe baza de date `dexonline` in format
`stardict`. Formatul `stardict` este recunoscut de `KoReader` (singurul
software open source pe care l-am gasit pentru e-reader-e).

Pornind de la postul
[acesta](http://filosofie.unibuc.ro/~solcan/wt/orto/stardex.html), am
descarcat baza de date de la `dexonline` si am transformat-o in format
`stardict`.

TLDR
========================================

Ca sa obtineti un format `stardict`, trebuie mai intai obtinut un format `gls`
din baza de date dexonline a datelor returnate de select-ul acesta:

```
select replace(l.form, "'", '') form, replace(d.htmlRep, "\n", "<br/>") html from Lexem l inner join EntryLexem el on l.id = el.lexemId
inner join Entry e on el.entryId = e.id
inner join EntryDefinition ed on e.id = ed.entryId
inner join Definition d on ed.definitionId = d.id where d.sourceId in (27, 21);
```

Formatul `gls` este destul de simplu. Pe prima linie se afla cuvantul cautat,
apoi pe urmatoarea se afla definitia si dupa o linie libera. Deasupra datelor
trebuie adaugat header-ul acesta (cu tot cu linii libere): 

```

#bookname=dexonline.ro
#version=3.0.0
#stripmethod=keep
#sametypesequence=h
#date=2018-01-21
#description=Acesta este o adaptare a bazei de date DEX online sub licenÈ›a GPL.

```

iar dupa aceea trebuie trecut prin `stardict-editor` pentru a obtine cele 3
fisiere `stardict`. Eu am folosit anumite software si anumite proceduri ca sa
obtin formatul `gls`, dar evident, din moment ce se lucreaza cu soft-uri libere
si formate text, puteti folosi orice software va e la indemana.

Cum sa-l folosesc
========================================

Cele 3 fisiere `dex.*` trebuie copiate in folderul de dictionare al koreader sau
al oricarui alt cititor care cunoaste `stardict`. Daca doriti, puteti reincepe
procesul pentru a obtine din nou baza de date si dictionarul.

Softuri folosite
========================================

* [Artix Linux](https://artixlinux.org/)
* [vim](https://www.vim.org)
* [Sql Workbench/j](http://www.sql-workbench.net/)
* [StarDict editor](https://aur.archlinux.org/packages/stardict-tools/)
* `dictd` (din pachetele `Artix Linux`).
* `MySql`

Pasi
========================================

## Instalati softurile necesare.

Atentie la `stardict-tools`. Pachetul este configurat sa depinda de
`libmysqlclient`, care nu mai exista acum in `Arch` sau `Artix Linux`. Trebuie
sa editati fisierul `PKGBUILD` si sa eliminati de acolo aceasta dependinta.
Probabil `libmysqlclient` este folosit de alte softuri din pachet,
`stardict-editor` (care ne trebuie noua) va merge.

## Descarcati baza de date `dexonline` de
[aici](https://dexonline.ro/static/download/dex-database.sql.gz)

## Incarcati-o in `mysql`.

Un exemplu poate fi vazut
[aici](https://wiki.dexonline.ro/wiki/Instruc%C8%9Biuni_de_instalare)

## Folosind `Sql Workbench/J`, exportati datele in format CSV

In documentatia `Sql Workbench/J` sunt explicate anumite comenzi specifice, ca
`wbexport` Le puteti gasi acolo. Eu am executat aceste comenzi:

```
wbexport -header=false -file=/tmp/rows.txt -delimiter=, -quoteChar="'" -quoteCharEscaping=none -quoteAlways=false -encoding=UTF-8;

select replace(l.form, "'", '') form, d.htmlRep html from Lexem l inner join EntryLexem el on l.id = el.lexemId
inner join Entry e on el.entryId = e.id
inner join EntryDefinition ed on e.id = ed.entryId
inner join Definition d on ed.definitionId = d.id
```

In urma acestei operatii, am obtinut datele in format text in fisierul
`/tmp/rows.txt`.

## Preparati fisierul text

Fisierul text obtinut trebuie transformat in format `.gls`. Puteti gasi
informatii despre format [aici](https://fileinfo.com/extension/gls).

Am editat fisierul cu `vim` si am facut urmatoarele schimbari:

* `:%s/\v'$//g` (am eliminat `'` de la sfarsitul fiecarei linii unde exista)
* `:%s/\v^([^,]+),'/\1,` (am eliminat `'` de dinaintea definitiilor)
* `:%s/\v^([^,]+),(.*)$/\1^M\2^M` (am separat lexemele de definitii si am
adaugat o linie noua dupa fiecare definitie)

Atentie la caracterele speciale: au fost obtinute apasand `Ctrl` si `V` si
apoi `Enter`. Am inlocuit in principiu textele gasite cu linii noi.

Acum datele sunt in format `.gls`.

Creati fisierul `dex.gls` cu urmatorul continut:

```

#bookname=dexonline.ro
#version=3.0.0
#stripmethod=keep
#sametypesequence=h
#date=2018-01-21
#description=Acesta este o adaptare a bazei de date DEX online sub licenÈ›a GPL.

-continut-rows.txt-modificat-
```

Atentie la liniile libere. Prima linie din fisier trebuie sa fie o linie
libera si dupa definitia fisierului (linniile care incep cu `#`) trebuie
inserat continutul modificat al `rows.txt`.

Salvati fisierul si inchideti `vim`.

## Creati dictionarul

Deschideti `stardict-editor`. In interfata, selectati jos in stanga formatul
`Babylon file` (nu `Tab file`, care este selectat initial). Apoi selectati
fisierul creat `dex.gls` cu optiunea `Browse` in partea din stanga sus. Apoi
apasati `Compile` in partea din dreapta jos. Daca totul a mers ok, ar trebui
sa obtineti un mesaj de genul acesta:

```
Building...
Over
wordcount: &lt;n>
Done!
```

unde `n` este numarul de cuvinte din dictionar.

Acum ar trebui sa aveti langa fisierul `dex.gls` cele 3 fisiere `stardict`
(`dex.dict.dz`, `dex.idx`, `dex.ifo`). Aceste 3 fisiere reprezinta dictionarul
in format `stardict`. Trebuie sa le copiati in functie de e-reader-ul folosit
in folderul indicat de [documentatia
`KoReader`](https://github.com/koreader/koreader/wiki/Dictionary-support)

