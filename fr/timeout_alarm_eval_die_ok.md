# Timeout: alarm eval die
Comment traiter **correctement** et de **manière générale** le problème de timeout d’une fonction.

Dans la première partie nous avons vu un historique des versions publiées, mais fausses, dans des livres de références sur Perl.

Dans cette seconde partie je vous propose une version correcte et générale de traiter ai meiux ce problème du timeout d'une fonction.



## Implémentation correcte et générale

```Perl
01 sub runWithTimeout {
02   my $func    = shift;
03   my $timeout = shift || 60;
04   my $result;
05
06   eval {
07     local $SIG{ALRM} = sub { die "timeout\n" };
08     alarm($timeout);
09     eval { $result = &{$func}() };
10     alarm(0);
11     die "TIMEOUT!\n" if $@ eq "timeout\n";
12     die $@ if $@;
13   };
14   $@ = ''  if $@ eq "timeout\n";
15   return $result;
16 }
```
## Explications

02: Le premier paramètre $func est une référence à la fonction à exécuter.

03: Le deuxième paramètre est le temps que l'on donne à $func pour faire son travail. 60 secondes par défaut.
 
04: $result servira à récupérer le résultat renvoyé par $func si elle se termine avant la fin du delai imparti.

06: Ce premier eval est là pour capturer le cas rare, mais possible, où l’alarme se déclenche après la sortie du deuxième eval (ligne 9) mais avant que alarm(0) ne prenne effet. Dans ce cas précis, malgré le déclanchement de l’alarme, la fonction à fait son travail.

07: On localise le gestionnaire du signal ALRM et on lui associe une fonction qui exécute un die avec  ‘timeout\n’ comme message. A la sortie du premier eval ce gestionnaire retrouvera sa valeur d’origine.

08: On enclenche l’alarme.

09: Le deuxième eval permet d’executer $func et de capturer toutes les causes de die pouvant apparaitre pendant l’exécution de $func.

10: alarm(0) annule l’alarme. Si on a quitté le deuxième eval avant la fin du délai l’alarme etant encore active elle peut se déclencher à tout moment.

11: Transforme l'eventuel die "timeout\n" due à l’alarme en un die "TIMEOUT\n" (on verra la raison plus bas)

12: Si necessaire on fait suivre toutes les autres causes de die sans les modifier.

14: Si on a quitté le premier eval à cause de ‘timeout’ alors on ignore cette raison. En effet le seul moyen de quitter le premier eval avec ‘timeout’ c’est quand l’alarme s’est déclenché après la sortie du deuxième eval mais avant la prise d’effet de alarm(0).  Dans ce cas la fonction a fait son travail et on doit ignorer ce timeout.

15: Le travail de $func a été fait dans les délais on retourne son résultat.


## usages

* Si la fonction à executer n'a pas de parametres :
  ```Perl
  $name = runWithTimeout(\&getCustomerName);
  ```

* Utiliser une fonction anonyme s'il faut passer des paramètres à la fonction :
  ```Perl
  $transacId = runWithTimeout( sub{transaction($client, $produits, $cb)}, 10*60);
  ```

* Traiter les conditions de retour:
  ```Perl
  $transacId = runWithTimeout( sub{transaction($client, $produits, $cb)}, 10*60);
  if ($@){
    # quelque chose s'est mal passé
    if ($@ eq "TIMEOUT!\n"){
      # un probleme de timeout
    }else{
      # un autre probleme
    }
  }else{
    # traiter les cas normaux
    if($trancId){
      # la transaction a abouti dans le delais
    }else{
      # la transaction a échouée de manière previsible dans le delais
    }
  }
  ```

## ATTENTION !


