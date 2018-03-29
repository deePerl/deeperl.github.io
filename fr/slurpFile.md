# Slurp

Tout a était dit sur le _slurping_ dans [c’est article](http://search.cpan.org/~drolsky/File-Slurp-9999.13/extras/slurp_article.pod) de l’auteur du module `File::Slurp`.
En resumé: vitesse de traitement mais risque d'une forte empreinte mémoire.

Voici cependant ma modeste contribution avec cette petite fonction **slurpFile()** qui s’adapte au contexte pour renvoyer tout le fichier dans un scalaire ou dans une liste de lignes.

```
sub slurpFile {
  local $/ = undef unless wantarray;
  open(my $fh, '<', $_[0]) || return undef;
  return <$fh>;
}
```

### Notes:
* On laisse volontairement le soin à Perl de fermer le fichier `$fh`.
* En principe on évite d'écrire `return undef;`, mais là on veut pouvoir distinguer un fichier vide d'un fichier impossible à ouvrir.

## Usage:
``` Perl
#  Scalar context: renvoi tout le fichier dans une chaine.
my $pwd = slurpFile('/etc/passwd');
say defined $pwd ? $pwd : $!;
```

``` Perl
# List context: revoi un tableau avec une ligne par élement
my @pwd = slurpFile('/etc/passwd');
my %usr_id  = map { (split /:/)[0,2] } @pwd;
say dumpit \%usr_id;
```

## En cas d'erreur

En cas d’erreur on exécute `return  undef ;`
- Si le contexte d’origine est scalaire on a donc un scalaire `undef`.
- Si le contexte d’origine est liste on a une liste dont l’unique élément est `undef`.

Si on avait juste utilisé `return ;`
- rien n’aurait changé pour le contexte scalaire 
- mais dans le contexte liste on aurait eu une liste vide, ce qui est indistinguable du fichier vide.

|  en resumé              | `return ;` | `return undef ;` |
| ----------------------- | ---------- | ---------------- |
| scalaire + fichier vide | ' '        | ' '              |
| scalaire + erreur       | `undef`    | `undef`          |
| liste + fichier vide    | ( )        | ( )              |
| liste + erreur          | ( )        | ( `undef`)       |

A vous de voir ce qui vous arrange. Il est toujours possible de tester `$!`
