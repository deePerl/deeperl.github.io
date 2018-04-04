
# dumpit

Voici un petit module qui permet de faire un dump d'un ou plusieur objets passés en réference.
Le coeur du travail est confié au module standard Data::Dumper.

``` perl
package dumpit;

use strict;
use warnings;
use v5.10;

use Data::Dumper;

use Exporter 'import';
our @EXPORT = ('dumpit');

sub dumpit {
  local $Data::Dumper::Useperl       = 1;
  local $Data::Dumper::Terse         = 1;
  local $Data::Dumper::Sortkeys      = 1;
  local $Data::Dumper::Indent        = 1;
  local $Data::Dumper::Useqq         = 1;
  local $Data::Dumper::Quotekeys     = 0;
  local $Data::Dumper::Deparse       = 1;
  local $Data::Dumper::Trailingcomma = 1;

  return Dumper($_[0]) if @_==1;

  if (@_ % 2) {
    say "ERROR: 1 ou 2n parametres!\n";
    return;
  }

  while (my($name, $ref) = splice(@_, 0, 2)) {
    say $name, ': ', Dumper($ref);
  }

  return;
}

1;
```

### USAGE

* Dans sa version un seul parametre dumpit() retourne le dump de l'objet.

  `my $a = dumpit(\%annuaire);`

* Dans sa version 2,4,6 ... 2n parametres il affiche le dump de l'objet
 
  `dumpit('Contacts' => \%annuaire);`  # on peut utiliser une virgule à la place de =>

  ```console
  Contacts: {
    jean => "+32 012345678",
    marie => "0612345678",
    pierre => "123 45678999"
    police => 18
  }
  ```

  `dumpit('' => \@a, 'Contacts' => \%annuaire);`

### ATTENTION
Le comportement de Data::Dumper est étrange quand il s'agit d'afficher des nombres:

`dumpit("Resultats" => [1, 2, -1, +2, 3.4 , "5"]);`
 
```console
Resultats: [
  1,
  2,
  -1,
  2,
  "3.4",             <-- quotes !
  5,                 <-- pas de quotes !
]
```
