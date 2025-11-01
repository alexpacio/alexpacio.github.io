Et sinon LinkedIn, si tu arrêtais de raconter n'imp sur les VM ?
Une VM en 2025 (et depuis plus de 10 ans) ce n'est plus un émulateur, c'est un objet qui, bien configuré (i.e. VirtIO à tous les étages), s'exécute à native speed. Une VM c'est un espace d'adressage séparé du host, possiblement également un noyau différent, qui garantit donc une réelle isolation des execitions. Le noyau en question peut booter en moins de temps que ton init.
Non, docker et compagnie m'isolent pas vraiment. docker et compagnie utilisent une forme de fork(), unshare(). Les failles du host sont directement exploitables dans le container.

---


Mon précédent billet d'humeur sur les "machines virtuelles" ayant eu un certain succès, je renchéris ici avec quelques explications un peu plus deep-dive en vulgarisant les concepts les plus techniques.

Déjà, pourquoi désormais je mets des guillemets à "machine virtuelle" ? Parce que la façon de démarrer un OS *différent* sur une machine physique n'a plus grand chose de virtuel, le CPU réalise une forme de context switch, c'est à dire qu'il sauvegarde son état à l'instant, avant de basculer sur l'execution des instructions de la "machine virtuelle", une fois ce switch effectué, le CPU (ou plutôt le coeur) va *exécuter* les instructions de la "VM" nativement, il n'y a PAS d'émulation si la VM est de la même architecture que le host.

Mettez ça en perspective maintenant, cela signifie que le noyau que vous êtes en train de démarrer pour booter votre VM n'est pas "interprété", "émulé" ou je ne sais quelle ânerie, non, le noyau invité est *exécuté*, comme un bête programme.
Et je dis bien "le noyau est en train de booter", je ne parle pas de bootloader, et pour cause, n'importe quel VMM (Virtual Machine Manager) récent sait *démarrer* le noyau -pratiquement- directement, en se branchant sur un entrypoint particulier (PVH boot).
Pas de grub, aucun bootloader. La légende de "lA vM qUi mEt dU tEmPs à dÉmArReR" en 2025 est un gros tas de flan.
Qemu par exemple, sait utiliser cette fonctionnalité via le paramètre `-kernel` auquel on passe le noyau à démarrer.

Ensuite, du point de vue de Qemu, l'OS que vous bootez n'est qu'un ensemble de threads dont le nombre est (au moins) égal au nombre de vCPUs que vous lui demandez d'affecter à la VM, il réservera une certaine quantité de mémoire qu'il mettra à disposition de cette dernière qui l'utilisera comme "mémoire physique" (GPA, pour Guest Physical Address).

Ce genre de deep dive vous intéresse ? Si oui dites le dans les commentaires et on continuera à réduire le temps d'execution d'une VM avec VirtIO, dont le design permet de s'affranchir de l'émulation des périphériques...


---

Dans l'épisode précédent, nous avons vu que de nos jours, les instructions d'une "machine virtuelle" au sein d'une même architecture ne sont justement pas virtuelles et sont pour la plupart directement exécutées par le CPU.

Notez le subtil "pour la plupart". Nous allons y revenir.

Il y a peu de temps, pendant un meeting "technique", quelqu'un lache : "oui c'est optimisé avec VirtIO". Aucun mot ne signifie mieux "je comprend rien à ce que je raconte" que le mot "optimisé".
VirtIO n'"optimise" rien du tout. VirtIO est une façon d'implémenter des pilotes virtuels en minimisant le temps que passe le CPU à sortir (VMEXIT) et rentrer (VMENTER) dans la machine virtuelle. En effet, comme nous l'avons vu dans l'épisode précédent, basculer du host vers le guest suppose un switch de contexte, et comme pour son pendant lors de l'execution d'un processus, ce switch coûte cher. Or, dans les anciennes implémentations de la virtualisation, les périphériques étaient émulés, cette émulation est réalisée par le CPU, mais du côté du host, plus particulièrement dans le VMM (Qemu, Firecracker, VirtualBox...), ce qui signifie par exemple que pour *chaque* paquet réseau reçu, pour *chaque* caractère écrit à l'écran, on VMEXIT/VMENTER i.e. sauvegarde puis restauration de contexte.

Un driver implémenté en utilisant VirtIO évite cet overhead en n'émulant pas du matériel, mais en mettant simplement en place une zone de communication directe entre le host et le guest.
Rappelez-vous, nous avons vu que le VMM, lorsqu'il démarre le système invité, lui met à disposition un bloc de mémoire (GPA, Guest Physical Address), cette zone mémoire est accessible par les deux mondes, et permet donc la communication entre eux.
Grace à cette technique, le coût d'un I/O est le même que celui d'aller lire une donnée en mémoire, un bête déplacement d'octets entre la mémoire et un registre du CPU, car finalement, qu'est-ce qu'un driver si ce n'est un programme qui fait transiter des informations d'un périphérique d'entrée / sortie, parfois jusqu'au CPU, parfois vers un autre périphérique.

C'est tout pour cette fois, je souhaite que ces posts restent courts, mais si vous désirez qu'on rentre un peu plus profondément dans les arcanes d'un driver VirtIO, manifestez-vous !