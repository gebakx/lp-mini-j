# Pràctica LP: G

Aquesta pàgina descriu la pràctica de GEI-LP (edició 2024-2025 Q2). En aquesta pràctica has d'escriure un intèrpret G; una versió simplificada de J (derivat d'APL) utilitzant Python, ANTLR i numpy.

APL va ser desenvolupat per Kenneth E. Iverson als anys 60. És un llenguatge funcional
molt especialitzat en manipulació de vectors i matrius i amb una forta notació matemàtica. El 1979 Iverson va rebre el premi Turing pel seu treball en APL.

APL és un llenguatge conegut per no utilitzar caràcters ASCII i ser molt concís, encara 
que pot semblar críptic per als no iniciats. 

El llenguate J és un derivat d'APL desenvolupat pel mateix Iverson als anys 90 que utilitza només caràcters ASCII.

L'expressió matemàtica $m=\sum_{i=1}^4(i+3)$ quedaria com:

- APL: `m ← +/3+⍳4` 

- J: `m =: +/ 3 + i. 4`

En aquesta pràctica haureu de treballar amb un subconjunt G del llenguatge J.

## G (mini J)

A continuació tens un breu resum dels conceptes bàsics de J. Pots trobar més informació en el llibre [Learning J](https://www.jsoftware.com/help/learning/contents.htm) i provar-lo en el [J playground](https://jsoftware.github.io/j-playground/bin/html2/). A més, aquesta secció també afita les construccions de G que caldrà implementar en aquesta pràctica. Has de dissenyar la gramàtica en ANTR per a que reconegui les diferents construccions que esmenta aquest document i utilitzar el [numpy](https://numpy.org/) mitjançant el visitor per interpretar el codi.

### Sintaxi bàsica

En J, el tipus principal són llistes que s'escriuen separades per espais i sense cap delimitador exterior:

```j
1 2 3
```

Les operacions es fan sobre llistes:

```j
1 1 1 + 1 2 3    NB. resultat: 2 3 4
```

Com pots veure els comentaris comencen per `NB.`.

En cas de que un dels operands sigui un escalar, opera l'escalar amb tot el vector:

```j
1 + 1 2 3    NB. resultat: 2 3 4
```

Però, si operem dos llistes amb diferent mida (except el cas anterior) dóna un error:

```j
1 1 + 1 2 3   NB. resultat: length error
```

Els operadors tenen tots la mateixa prioritat i les expressions s'avaluen de dreta a esquerra:

```j
5 + 2 * 3    NB. resultat: 11

5 * 2 + 3    NB. resultat: 25
```

Podem utilitzar parèntesis per control·lar el comportament:

```j
(5 * 2) + 3    NB. resultat: 13
```

El signe dels nombres negatius és el guió baix:

```j
_1 * 2 3    NB. resultat: _2 _3
```

### Operacions bàsiques

A més de la suma, J té altres operacions bàsiques com:

- Restar: `5 - 2` (resulta en `3`). També és l'operador unari.
- Multiplicar: `2 * 3` (resulta en `6`)
- Dividir: `6 % 2` (resulta en `3`). En J correspon a la divisió real, però en G us demanem l'entera per treballar només amb enters.
- Residu: `2 | 7` (resultat en `1`). Vigileu que els operands van al revés.
- Potència: `2 ^ 3` (resultat en `8`).

Els operadors artimètics requerits són `+`, `-`, `%`, `|` i `^`. Aquests operadors numèrics s'anomenen _verbs_ en J.

### Booleans i operadors relacionals

En J, `1` representa `true` i `0` representa `false`. 

Els operadors relacionals requerits són: `>`, `<`, `>=`, `<=`, `=`, `<>`.

### Altres operacions

- `]` és la funció identitat:
  
  ```j
  ] 1    NB. resultat: 1
  ```

- `,` correspon a la concatenació:
  
  ```j
  1 , 2 3    NB. resultat: 1 2 3
  ```

- `#` unari ens retorna la mida d'un vector:
  
  ```j
  # 1 2    NB. resultat: 2
  ```

- `#` binari fa un filtre amb una màscara:
  
  ```j
  1 0 1 0 # 1 2 3 4    NB. resultat: 1 3
  ```

- `{` serveix per accedir a element per índex:
  
  ```j
  0 2 { 2 3 4   NB. resultat: 2 4
  ```

- La funció `i.` ens retorna un vector amb els n primers naturals:
  
  ```j
  i. 4    NB. resultat: 0 1 2 3
  ```

- Si afegim `:` a qualsevol operador binari el converteix en unari i utilitza el seu operand dues vegades:
  
  ```j
  +: 1 2 3    NB. resultat: 2 4 6
  ```

- `/` fa un *fold*:
  
  ```j
  + / 1 2 3    NB. resultat: 6
  ```

- flip:
  
  ```j
  7 | ~ 2    NB. resultat: 1
  ```

J anomena _operadors_ a les funcions d'ordre superior _adverbis_ als operadors unaris.

### Variables

L'assignació es fa amb el símbol `=:`:

```j
x =: 1 2 3
1 + x        NB. resultat: 2 3 4
```

### Definició de funcions

Les funcions les definim amb l'assignació:

```j
square =: *:
square 1 2 3 4    NB. resultat: 1 4 9 16
```

La identitat:

```j
mod2 =: 2 | ]
mod2 i. 4    NB. resultat: 0 1 0 1
```

I la composició:

```j
eq0 =: 0 = ]
parell =: eq0 @: mod2
parell i. 6    NB. resultat: 1 0 1 0 1 0
```

Més exemples:

```j
square =: *:
square 1 + i. 3    NB. resultat: 1 4 9

mod2 =: 2 | ]
eq0 =: 0 = ]

eq0 mod2 i. 6    NB. resultat: 1 0 1 0 1 0

parell =: eq0 @: mod2
parell i. 6    NB. resultat: 1 0 1 0 1 0

parell =: 0 = ] @: 2 | ]
parell i. 6    NB. resultat: 1 0 1 0 1 0

inc =: 1 + ]
test =: +/ @: inc @: i.
test 3    NB. resultat: 6
```

**Nota**: només es demanen funcions com les que surten en els exemples.

## La vostra feina

Heu d'escriure un intèrpert de G utilitzant ANTLR, numpy i Python. Aquest intèrpret ha de ser capaç de llegir i avaluar expressions de G, així com definir i cridar funcions. Heu d'utilitzar ANTLR per escriure la gramàtica i els visitadors necessaris. Cal que la vostra implementació sigui compatible, com a mínim, amb la sintaxi i les característiques de J descrites anteriorment.

El vostre programa s'ha de preparar amb un cop de `make`. Llavors, el vostre intèrpret s'ha d'invocar amb la comanda `python3 g.py` tot passant-li com a paràmetre el nom del fitxer que conté el codi font (l'extensió dels fitxers per programes en scheme és `.j`). Per exemple:

```bash
make
python3 g.py programa.j
```

El programa sempre ha de començar per la funció `main`. L'entrada-sortida ha de ser via stdin/stdout. Així es podran utilitzar operadors de redirecció i *pipes*:

```bash
python3 g.py programa.j > sortida.txt
```

### Jocs de proves

El vostre projecte ha d'incloure jocs de proves que demostrin que el vostre intèrpret funciona correctament. Aquests jocs de proves han de ser fitxers amb extensió `.j` que continguin programes vàlids en G. La qualitat (però no la quantitat) dels jocs de proves serà un factor important en l'avaluació de la pràctica. Els jocs de proves poden venir acompayats de fitxers de sortida (`.out`) per facilitar la seva repetició.

### Llibreries

Utilitzeu `ANTLR` per escriure la gramàtica i l'intèrpret. Podeu utilitzar lliurament qualsevol llibreria **estàndard** de Python. No podeu usar cap altra llibreria no estàndard.

### Errors

Si el programa en G conté errors sintàctics, cal reportar-ho.

En canvi, per senzilla, en aquesta pràctica, suposarem que no es dónen mai errors semàntics ni errors de tipus. En cas de donar-se, l'efecte del programa és indefinit.

Igualment, suposarem que mai es donen errors d'execució (com ara divisions per zero, agafar el cdr d'una llista buida, ètc). En cas de donar-se, l'efecte del programa és indefinit.

## Lliurament

Heu de lliurar la vostra pràctica al Racó. Només heu de lliurar un fitxer ZIP que, al descomprimir-se generi:

- Un fitxer `README.md` que documenti el vostre projecte.
  
  - vegeu, per exemple, https://www.makeareadme.com/.

- Un fitxer `Makefile` tal que, quan s'executi `make`, es crein els fitxers necessaris per executar el vostre projecte.

- Un fitxer `g.g4` amb la gramàtica del LP.

- Un fitxer `g.py` amb el programa principal de l'intèrpret.

- Més fitxers `.py` amb les classes, visitadors i funcions auxiliars.

- Jocs de proves en fitxers `.j` amb sortides en fitxers `.out`.

- Res més. 

Observacions:

- Els vostres fitxers de codi en Python han de seguir les regles d'estı́l PEP8, tot i que podeu oblidar les restriccions sobre la llargada màxima de les lı́nies. L'ús de tabuladors en el codi queda prohibit (zero directe).

- El termini de lliurament és el **dilluns 2 de juny a les 08:00**.

- Per evitar problemes de còpies, no pengeu el vostre projecte en repositoris públics.

- El vostre lliurament no ha d'incloure els fitxers que genera ANTLR, aquests s'han de crear via `make`.

- Si no heu realitzat alguna part de la pràctica, o sabeu que aquest té algun error en alguna part, deixeu-ho escrit al `README.md`.

## Avaluació

L'avaluació de la vostra pràctica tindrà en compte diversos aspectes clau, entre els quals es destaquen els següents:

1. **Qualitat de la gramàtica**: Es valorarà la qualitat de la gramàtica ANTLR, incloent la seva _completitud_, _precisió_, _concisió_ i _robustesa_. La gramàtica ha de ser capaç de reconèixer correctament els programes i les expressions de Mini Scheme, així com de manegar els diferents literals i operadors que es poden realitzar en aquest llenguatge. 

2. **Qualitat del codi**: S'examinarà el codi font tenint en compte diversos factors, com ara la _correctesa_, la _completitud_, la _llegibilitat_, l'_eficiència_, el _bon ús dels identificadors_, ètc. També es tindrà en compte la _bona estructuració_ del codi, és a dir, l'ús adequat de funcions, classes, mòduls i altres elements que afavoreixin la redacció, la comprensió i el manteniment del codi a llarg termini. Es valorarà negativament l'ús de funcions llargues o poc clares, funcions incomprensibles sense especificació, la presència de codi duplicat o innecessari, acoblament fort, variables globals o atributs de classe erronis, comentaris excessius, i altres pràctiques nocives. Per aquesta pràctica, l'eficiència és secundària, però no s'admetran disbarats.

3. **Qualitat de la documentació**: S'analitzarà la documentació del projecte, amb especial atenció a la seva _claredat_, _precisió_ i _completesa_, alhora que la seva _concisió_. La documentació hauria de descriure adequadament el funcionament del codi, les seves funcions i característiques principals, així com qualsevol altre aspecte rellevant que faciliti la comprensió i l'ús del projecte. La documentació també ha de deixar clares les decisions de disseny preses.

4. **Qualitat dels jocs de proves**: Es valorarà l'existència, la _cobertura_ i la _fiabilitat_ dels jocs de proves dissenyats per verificar el correcte funcionament del codi. Els jocs de proves han de ser suficientment amplis i variats per garantir que el codi respon de manera adequada a diferents situacions i casos d'ús. Tanmateix, els jocs de proves han de ser _limitats, _concisos_ i _eficaços_, evitant la redundància i la repetició innecessària. Alhora, el propòsit dels jocs de proves ha de ser _fàcil de comprendre_. Han de ser fàcils d'_executar_, i han de proporcionar una _sortida clara_ que permeti identificar ràpidament qualsevol problema o error.

En definitiva, és important recordar que es tracta d'un projecte de programació i, per tant, s'espera que el codi segueixi _bones pràctiques de programació_. 
