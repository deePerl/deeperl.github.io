# Le cache

Des centaines de livres et des dizaines de milliers d’articles ont été écrit sur les algorithmes de tri. Ceci ne sera pas un article de plus sur ce sujet. Je voudrais juste parler de ce que certains appellent, avec un certain culte de la personne typique au monde Perl, « Schwartzian Transform ». Sous ce nom se cache une technique très simple : le cache.

> Une legère variante de cette technique est nommée « [Guttman-Rosler Transform](./sort-oid) ».
Quand je vous le dit qu'il y a un culte de la personne...

# Le problème
On a une liste d’objets que l’on veut trier. Mais l'information sur laquelle le critère de tri doit s’appliquer n’est pas explicite, il faut la calculer. Autrement dit ce n’est pas un entier ou une chaine de caractères deja présent dans les objets à trier.

Par exemple trier des adresses ip d’après l’AS (Autonomous System) auquel elles appartiennent et le pays où elles sont allouées.

# Solution naïve
```Perl
@resultat = sort { computeKey($a) cmp computeKey($b) } @liste; # cmp ou <=> suivant le type de computeKey()
```
Chaque fois que l'algorithme de tri devra comparer deux enregistrements il calculera la clé qui leur correspond et comparera ces clés. On comprend immédiatement que quel que soit l’algorithme de tri utilisé la clé de chaque élément sera calculée plusieurs fois ce qui risque de prendre beaucoup de temps. (Enormément de temps s’il y a un accès à un fichier, une base de donnée, une connection réseau etc.., à chaque fois).

# Solution classique
Tout développeur sérieux envisagera immédiatement de mettre en cache le résultat de chaque computeKey($a) et de l’associer à $a. Dans bien des cas il n’est pas possible de modifier $a pour y inclure le résultat de computeKey($a). L’association doit être externe à $a. 

> **Attention**: L’usage d’un hash est un piège dans lequel il ne faut pas tomber ! En effet si deux éléments ont la même clé le hash n’en retiendra qu’un seul.
>
> **Rappel**: $a et $b sont des références à des éléments du tableau à trier.

La bonne méthode est de faire un tableau de paires. (En fait: un tableau de références à des tableaux de deux éléments). Un élément de la paire est le résultat du calcul de computeKey($x) l’autre est $x.

Une fois la liste de paires construite ainsi
```Perl
# phase d'association
@paires = map { [computeKey($_), $_] } @liste ; 
```
on pourra écrire le tri de cette façon :
```Perl
# phase de tri
@pairesTriees = sort { $a->[0] cmp $b->[0]} @paires;  # cmp ou <=>
```
Sauf que ce que l’on a trié ce sont les paires… Il faut en extraire les références aux elements d'origine.
```Perl
# phase de dissociation 
@resultat = map { $_->[1] } @pairesTriees;
```

> **Attention**: à la fin de ces 3 étapes on a en mémoire : ```@liste```, ```@paires```, ```@pairesTriees``` et ```@resultat```. Soit au moins 4 fois plus qu'au depart. Cela peut être un problème.

Cet algorithme association/tri/dissociation n’est pas specifique à Perl, et était connu bien avant Perl. On peut la mettre en œuvre dans pratiquement tous les langages. Là où ça devient plus Perlish c’est quand on enchaîne ces trois opérations en une seule.
```Perl
@resultat = map { $_->[1] }   sort { $a->[0] cmp $b->[0] }   map { [computeKey($_), $_] }   @liste ;
```

Comme souvent en Perl ceci se lit/comprend de droite à gauche.

Le seul avantage de cette fusion en une seule ligne (*en plus d’impressionner ou dégoûter le débutant*) est le nettoyage implicite de l’espace mémoire. Les tableaux intermédiaires sont desalloués à la fin de l’instruction.

Une fonction pour concentrer et documenter tout cela:

```Perl
# map-sort-map
sub msm_sort {
  my($unsorted, $computeKey) = @_ ;
  return map { $_->[1] }  
         sort { $a->[0] cmp $b->[0] }  # cmp ou <=> suivant ce que retourne $computeKey->()
         map { [$computeKey->($_), $_] } 
         @{$unsorted} ;
}
```

# Dénomination
Cet idiome était jadis appelé decorate-sort-undecorate (en tout cas c’est sous ce nom que je l’ai étudié quand j’étais à l’université au début des années 80), je l’ai appelé association-tri-dissociation, il correspond à l'idiome Perl map-sort-map et est aussi appelé « Schwartzian Transform » (ST pour les intimes) suite à un post effectué en 1994 par Monsieur Randal L. Schwartz  qui a été repris et mis en valeur par Tom Chistiansen. C'est ça être jeune: on s'imagine tout inventer en ignorant ce que les vieux on déja fait ;-)
