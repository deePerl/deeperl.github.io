# Operateurs: <=> et cmp

Perl n’a que deux types de données scalaires : les nombres et les chaines de caractères. Pour chacun de ces types il y a de nombreux opérateurs. Parmi eux il y en a deux qui on une  ‘beauté’ spéciale...

## -1 0 +1
Il s’agit du _spaceship operator_, c’est-à-dire l’operateur `<=>` et de son frère `cmp`. `<=>` s’applique aux nombres, `cmp` s’applique aux chaines. Leur point commun est de retourner -1, 0 ou +1 suivant que l'opérande de gauche est inférieur, égal ou supèrieur à l'opèrande de droite.

La beauté de ces opérateurs vient de ce qu’une fois implémentés les 6 autres opérateurs de comparaison en découlent.

## Théorie
Pour voir cette beauté prenons l’opérateur `<=>` (la logique est la meme avec `cmp`)

Si on pose que `op` représente n’importe lequel des 6 operateurs de comparaison sur les nombres : `<` , `<=` , `==` , `!=` , `>=` , `>` alors on peut dire que

> **`a op b` donne le même résultat que `(a <=> b) op 0`**

Exemple:

> `a >= b` donne le même résultat que `(a <=> b) >= 0`
>
> `a != b` donne le même résultat que `(a <=> b) != 0`
>
>	etc…

Appliqué aux chaines ça donne:

> `a ge b` donne le même résultat que `(a cmp b) >= 0`
>
> `a ne b` donne le même résultat que `(a cmp b) != 0`
>
>	etc…

C’est sympa, mais comme sur les deux types de base (nombres et chaines) on a déja les opérateurs ce n'est pas vraiment utile.

## Pratique

Imaginons que nous dévions écrire un module manipulant un type spécial de donnée et que nous devions implémenter les fonctions de comparaisons comme pour les chaines (eq, ne, lt, ). Le but étant de pouvoir écrite ce type de code
```Perl
if ($a->eq($b)) { ... }
if ($a->le($b)) { ... }
sort { $a->cmp($b) } @liste ;
```

Et bien réjouissons-nous, nous n’avons vraiment à écrire que la fonction `cmp` !

```Perl
sub cmp {
  my ($a, $b) = @_;
  # logique pour retourner -1 0 +1
  # en fonction de $a et $b
}
```

Pour les 6 autres fonctions il faut juste écrite ces 6 lignes :
```Perl
sub lt { return ($_[0]->cmp($_[1])  < 0 }
sub le { return ($_[0]->cmp($_[1]) <= 0 }
sub eq { return ($_[0]->cmp($_[1]) == 0 }
sub ne { return ($_[0]->cmp($_[1]) != 0 }
sub ge { return ($_[0]->cmp($_[1]) >= 0 }
sub gt { return ($_[0]->cmp($_[1])  > 0 }
```
## C'est pas beau !
7 fonctions délivrées pour une seule à écrite, documenter, tester et maintenir.
C’est pas beau !

**Mais il y a plus fort** : Mettez ces 6 fonctions dans la classe de base et implémentez `cmp` dans les classes dérivées. Pour chaque `cmp` implementé vous avez les 6 autres fonctions gratuitement !

> Note: Les mecs de C++ viennent juste de découvrir ça et se proposent d'ajouter l'opérateur `<=>` au standard C++20. Vous codez `operator<=>` pour votre type et le compilateur génère tout seul les 6 autres opérateurs. Les développeurs C++ deviennent aussi écononoment que les développeurs Perl !
