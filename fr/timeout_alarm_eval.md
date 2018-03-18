# Timeout: alarm eval et die
Nous allons voir comment traiter **correctement** et de **manière générale** le problème de timeout d’une fonction.

Le premier intérêt de ce qui suit est didactique car deux livres notoires ont publiés des solutions fausses à ce problème.
-	La première fois en 1997 dans « Advanced Perl programming ». [5.6 Using Eval for Time-Outs]
-	La deuxième fois en 1998 dans « Perl Cookbook ». [16.21. Timing Out an Opération]
- La presque bonne version ne viendra qu’en 2003 avec la seconde édition de « Perl Cookbook ».

Le deuxième intérêt, plus pratique, est que la solution qui sera proposée ici est généraliste et permet de mettre sous contrôle d’un timeout n’importe quelle fonction sans avoir à la modifier (ce qui est très pratique si la dite fonction est dans un module que vous ne métrisez pas).

## Principes
L’idée de base est la suivante :
*	On va utiliser `alarm()` pour être averti quand le délais sera atteint.
*	On annulera l’alarme si on finit avant son déclenchement.
*	L’action déclenchée par l’alarme sera un `die()`.
*	Pour que ce `die()` n’arrête le programme dans son ensemble on encapsulera la fonction à chronométrer dans un `eval { }` ce qui limitera la portée du `die()`.

Mais comme toujours avec les signaux les pièges sont dans les détails, et les développeurs les plus réputés se sont fait piéger...

## Première implémentation (publiée mais fausse)

_Advanced Perl Programming (First Edition, August 1997) - 5.6 Using Eval for Time-Outs_

``` perl{.line-numbers}
$SIG{ALRM} = \&timed_out;
eval {
    alarm (10);
    $buf = <>;
    alarm(0);           # Cancel the pending alarm if user responds.
};
if ($@ =~ /GOT TIRED OF WAITING/) {
    print "Timed out. Proceeding with default\n";
    ....
}

sub timed_out {
    die "GOT TIRED OF WAITING";
}
```

## Deuxième implémentation (presque juste)

_Perl Cookbook (**First** Edition, August 1998) - 16.21. Timing Out an Operation_

``` perl{.line-numbers}
$SIG{ALRM} = sub { die "timeout" };

eval {
    alarm(3600);
    # long-time operations here
    alarm(0);
};

if ($@) {
    if ($@ =~ /timeout/) {
                            # timed out; do what you will here
    } else {
        alarm(0);           # clear the still-pending alarm
        die;                # propagate unexpected exception
    }
}
```
En voyant cette deuxième version on comprend où est la préoccupation : 
que se passe-t-il si la longue l’opération exécute un `die` ? 
Et bien on sort du `eval` avant la fin du timeout et donc avec toujours l’alarme active.
Il faut donc désactiver cette alarme avec `alarm(0)`.

Mais la solution est incomplète car l’alarme peut se déclencher à tout moment, 
par exemple pendant le test `if ($@ =~ /timeout/)` et l’exécution du die arrête le programme.

## Troisième implémentation (toujours fausse)

_Perl Cookbook (**Second** Edition, August 1998) - 16.21. Timing Out an Operation_

``` perl{.line-numbers}
eval {
    local $SIG{ALRM} = sub { die "alarm clock restart" };
    alarm 10;                   # schedule alarm in 10 seconds
    eval {
          ########
          # long-running operation goes here
          ########
    };
    alarm 0;                    # cancel the alarm
};
alarm 0;                        # race condition protection
die if $@ && $@ !~ /alarm clock restart/; # reraise
```
Ici on a une tentative plus pro (mais toujours fausse)
-	`SIG{ALRM}` est localisé
-	Le premier `eval` sert à se protéger du cas où on sort du deuxieme eval suite à un `die` autre que celui due à l’alarme et donc avec l’alarme active et que cette alarme s’exécute entre la sortie du deuxieme block `eval` et le premier `alarm 0`.

Le **gros** problème c’est que dans l’immense majorité des cas on sortira du premier eval normalement et que `$@` sera une chaine vide et on aura perdu la cause de la sortie du deuxième `eval`.

Enfin, le deuxième `alarm 0` est superflue car on ne peut pas sortir du premier `eval` avec l’alarme active.

## Mon implémentation (juste et générale)
voir ici...
