# ST vs GRT
Nous avons vue la technique « Schwartzian Transform » (ST)  voici la technique « Guttman-Rosler Transform », ou GRT pour les intimes. (Messieurs Guttman et Rosler devaient être jaloux que Monsieur Schwart ai son nom associé à une technique plus vielle que lui, ils ont donc fait une variante ...) 

Cette variante est simple: GRT c’est comme ST mais le ```sort``` n’a pas de bloc de code. 

Cela signifie que le tri se fait sur un tableau ne contenant que des scalaires comme des noms, des adresses etc…  mais que ces valeurs scalaires ne se présentent pas sous une forme triable. On effectue donc sur chaque valeur une opération réversible qui les rend triables, on les tri, puis on annule la première opération. Ca rappelle donc le *association-tri-dissociation* de ST. Ici c'est plutot *modification-tri-restauration*.

# Exemple simple
On a des chaines contenant  «Prénom Nom âge» mais on veut trier sur les noms.

On remplace chaque «Prénom Nom âge» par «Nom Prénom âge», on fait le tri, puis on restitue l’ordre d’origine «Prénom Nom âge».  Comme le tri s’effectue sur des chaines de caractères il suffit  d’utiliser ```sort``` dans sa forme la plus simple.

Le raisonnement qui sous-tend GRT est que
-	```sort``` est plus rapide que ```sort { }``` car tout s’effectue au niveau du code C sans aller-retour avec le niveau Perl.
-	La création des paires ```[objet, key(objet)]``` peut être couteuse en mémoire.

# Exemple réel: trier des OID
Trier des OID utilisés par SNMP pour designer des objets dans la MIB.

Des oid écrit sous forme numérique ressemblent à ceci : 1.3.6.1.2.1.25 ou 1.3.6.1.2.1.2.2.1.2 ou 1.3.6.1.4.1.2636.3.1.13.1.5.12.3.0.0 ou 1.3.6.1.4.1.277.1.7.2.1.186

## version ST
```Perl
sub oid_sort_st {
  my $oids = shift;
  return map  { $_->[0] }
         sort { $a->[1] cmp $b->[1] }
         map  { [$_, pack('N*', (split(m/\./, $_)))] } 
         @{$oids};
}
```

## version GRT
```Perl
sub oid_sort_grt {
  my $oids = shift;
  return map  { $_ =~ s/\b\d//g; $_}
         sort
         map  { (my $oid = $_) =~ s/\b0*(\d+)/(length $1).$1/eg; $oid}  # voir note
         @{$oids}
}
```

> Note: à partir de Perl 5.14 on peut utiliser l'option ```r``` de l'operateur ```s///``` pour écrite simplement:
```Perl
map  { $_ =~ s/\b0*(\d+)/(length $1).$1/reg }  # perl 5.14
```

Le truc de cette méthode consiste à ajouter devant chaque index sa longueur 13 --> 213. On transforme "1.23.456.78.9" en "11.223.3456.278.19". On effectue le tri sur ces oid transformé, puis on retire ces digits ajoutés. Note: Ce truc marche tant que chaque index de l'OID fait moins de 10 digits, ce qui est toujours le cas dans la réalité.

## Benchmark

Malheureusement pour Guttman et Rosler les performances, dans ce cas, ne sont pas au rendez-vous en termes de temps d’exécution.

```Perl
use strict;
use warnings;

use Benchmark qw(:all);
use Data::Dumper;


sub oid_sort_st {
  # ... cf ci-dessus
}

sub oid_sort_grt {
  # ... cf  ci-dessus
}

my @unsorted = (
  '11.22.33.44.05.66.77.88.99',
  '11.22.33.44.51.66.77.88.99',
  '11.22.33.44.15.66.77.88.99',
  '11.22.33.44.01.66.77.88.99',
  '2.3.4.5.6.7.8.9.10.11.12.13.14.15.16',
  '2.3.4.5.6.7.8.9.10.11.12.13.14.15',
  '12.3.4.5.6.7.8.9.10.11.12.13.14.15.16',
  '21.3.4.5.6.7.8.9.10.11.12.13.14.15.16',
  '1.1.3.4.5.6.7.8.9.10.11.12.13.14.15.16',
  '1.11.3.4.5.6.7.8.9.10.11.12.13.14.15.16',
  '1.121.3.4.5.6.7.8.9.10.11.12.13.14.15.16',
);

my (@sortedByST, @sortedByGRT);

cmpthese( -1, { 
	st  => sub { @sortedByST  = oid_sort_st(\@unsorted) }, 
	grt => sub { @sortedByGRT = oid_sort_grt(\@unsorted)}, 
} ) ;

print Dumper \@sortedByST;
print Dumper \@sortedByGRT;
```
Les résultats montrent qu'ici la méthode GRT est entre 3 et 4 fois plus lente que ST.

```console
       Rate  grt    st
grt  3516/s   --  -71%
st  12274/s 249%   --
```

Ce resultat n'est pas une règle. Cela montre juste qu'il ne faut pas ce baser sur des principes mais sur des tests...
