---
layout: post
title: A better bash random password generator
date: 2010-04-21 14:03
disqus_identifier: 538694787
---

If you're into writing [bash](http://en.wikipedia.org/wiki/Bash_(Unix_shell)) scripts you've probably read the [Bash Advanced Scripting Guide](http://tldp.org/LDP/abs/html/) over at The Linux Documentation Project. If you made it all the way through to "Appendix A" you'll find a [contributed script for generating random passwords](http://tldp.org/LDP/abs/html/contributed-scripts.html#PW). This script works pretty good but I've made some changes to enhance it since I use it daily and needed some extra bang for my buck.

First the matrix of characters that is used when selecting a random one has had symbols added to it, but more importantly it too has been completely randomized to ensure that there is a better sample taken.

Secondly you can now specify both the length of the password as well the number of passwords generated. The length defaults to 8 and the number defaults to 1. To change either of them you pass them in as the first and second arguments respectively.

Without further adieu, here's the new function. Copy it into your ~/.bashrc file and then source it to activate.

{% highlight bash %}
randompass() {
        MATRIX="HpZld&xsG47f0)W^9gNa!)LR(TQjh&UwnvP(tD5eAzr6k@E&y(umB3^@!K^cbOCV)SFJoYi2q@MIX8!1"
        PASS=""
        n=1
        i=1
        [ -z "$1" ] && length=8 || length=$1
        [ -z "$2" ] && num=1 || num=$2
        while [ ${i} -le $num ]; do
                while [ ${n} -le $length ]; do
                        PASS="$PASS${MATRIX:$(($RANDOM%${#MATRIX})):1}"
                        n=$(($n + 1))
                done
                echo $PASS
                n=1
                PASS=""
                i=$(($i + 1))
        done
}
{% endhighlight %}

Examples:

<div class="highlight">
<pre><code>(#518:1u:0s) evan@borgstrom[/home/evan]: randompass
cU9gu4j@

(#519:1u:0s) evan@borgstrom[/home/evan]: randompass 16
whr^CHz@G7DoSd^A

(#520:1u:0s) evan@borgstrom[/home/evan]: randompass 8 5
)0w@gU!v
hPn1(Yh!
@(@w^6h^
J!^IRwGQ
@DM)8cc!

(#521:1u:0s) evan@borgstrom[/home/evan]: randompass 32 10
VV)Db3f20t@@3Hhr&jasuj0tj@!n&WeL
bCsOquZUaed(J(gtXh!v6VJCagDFAglU
0s1EXo(S7^!A@6an^JyV^tDHsD31L7z!
iZ1HPZBGQDF!Wn7512(I)uMJy82)n(1y
miWfu))bMwf&fy!!Z(&0e)QEi9B0@)16
7hxsmDPBupTPs(@Sj@CMY@)!VsPci)1b
tI)8@MJ2flDaQy85(iM)T1jXL4Q3NS(^
!80tzfXH&b@0OHA)PSf)cHb1)9t(7l(V
&@0dU(ld1^eB!j)LUFsJuv(Q2p(IDb^J
ydaY)((X4^&lF@hSuSR^LI^5aRCehdQh</code></pre></div>
