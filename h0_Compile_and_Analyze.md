# h0: Compile and Analyze
**Päivämäärä:** 20.8.2026  
**Ympäristö:** Xubuntu 24.04 LTS (VirtualBox)  
**Tekijä:** Valtteri Vuokila



# 1. Yhteenveto
Kirjoitin perus Hello World-tyylisen C-koodin(GeeksforGeeks), jossa tulostettiin vain "Terve". Analysoin syntynyttä binääriä Linuxin peruskomennoilla (file, strings, ldd ja objdump). Apuna ja lisäopetuksena käytin Googlen Gemini (Flash 3.6) -kielimallia, sillä yksinään nuo tulosteet eivät näin aloittelijan silmiin paljoa sanoneet. Lopputulemana oli melkoisesti lisää ymmärrystä valmiin tiedoston "tirkistelystä". Enemmän tässä tulivat C/C++ ja assembly tutuiksi kuin itse "hakkerointi", mutta oletettavasti näin oli tarkoituskin, koska molemmat ovat varmasti pakollisia tai ainakin erittäin hyödyllisiä taitoja käänteismallinnuksessa (reverse engineering).



## 2. Ohjelman kääntäminen

Lähdekoodi käännettiin `g++`-kääntäjällä nimellä `testi`:

```
g++ test.c -o testi

```

testasin myös huvikseni tallentaa saman c koodin c++ koodina ja se toimi tismalleen samalla tavalla (hyvä yhteensopivuus näemmä tai compiler teki taikansa oikein)

## 3. Binaarin analyysi ja havainnot
Analyysit täysin "aloittelijahakkerin" silmin.

### A. Tiedostotyyppi (`file`)

Ajettu komento ja komennon tuloste:

```
v@v-VirtualBox:~/koulu$ file testi
testi: ELF 64-bit LSB pie executable, x86-64, version 1 (SYSV), dynamically linked, interpreter /lib64/ld-linux-x86-64.so.2, BuildID[sha1]=86245413b58887330d1cd051af6d92d18b63961d, for GNU/Linux 3.2.0, not stripped

```

**Löydökset ja pohdinta:**

Tässä huomio kiinnittyi kohtaan "not stripped", tämä tarkoittaa ilmeisesti sitä, että koodissa olevia funktioiden nimiä (kuten main) ei ole piilotettu tai poistettu, vaan ne näkyvät edelleen. (Stackoverflow)


### B. Merkkijonot binaarissa (`strings`)

Ajettu komento ja komennon tuloste:

```
v@v-VirtualBox:~/koulu$ strings testi
/lib64/ld-linux-x86-64.so.2
puts
__libc_start_main
__cxa_finalize
libc.so.6
GLIBC_2.2.5
GLIBC_2.34
_ITM_deregisterTMCloneTable
__gmon_start__
_ITM_registerTMCloneTable
PTE1
u+UH
Terve
...

```

**Löydökset ja pohdinta:**

Yllätyin, että Terve löytyi suoraan tulostuksesta. Eli vaikka koodi käännettiin binaariksi, teksti ei mennyt suoriltaan salatuksi. Voisin kuvitella tuon olevan melkoinen tietoturvariski jos esim jonkin API-avaimen tai salasanan olisi "kovakoodannut" (.env systeemi käsittääkseni kyllä hoitaa tämän ja sen pitäisi olla joka koodarille perus kauraa)

---

### C. Kirjastoriippuvuudet (`ldd`)

Ajettu komento ja komennon tuloste:

```
v@v-VirtualBox:~/koulu$ ldd testi
	linux-vdso.so.1 (0x0000768610b38000)
	libc.so.6 => /lib/x86_64-linux-gnu/libc.so.6 (0x0000768610800000)
	/lib64/ld-linux-x86-64.so.2 (0x0000768610b3a000)

```

**Löydökset ja pohdinta:**

Tästä pystyin päättelemään että joku linuxin perus paketti käytössä. Muutoin oli melko hyödytöntä tietoa näin aloittelijalle.



### D. Disassemblointi ja assembly-koodi (`objdump`)

Ajettu komento ja komennon tuloste:

```
v@v-VirtualBox:~/koulu$ objdump -d -M intel testi | grep -A 20 "<main>:"
0000000000001149 <main>:
    1149:	f3 0f 1e fa          	endbr64
    114d:	55                   	push   rbp
    114e:	48 89 e5             	mov    rbp,rsp
    1151:	48 8d 05 ac 0e 00 00 	lea    rax,[rip+0xeac]        # 2004 <_IO_stdin_used+0x4>
    1158:	48 89 c7             	mov    rdi,rax
    115b:	e8 f0 fe ff ff       	call   1050 <puts@plt>
    1160:	b8 00 00 00 00       	mov    eax,0x0
    1165:	5d                   	pop    rbp
    1166:	c3                   	ret


```

**Löydökset ja pohdinta:**

Tässä kohtaa pitikin jo turvautua Geminiin enemmän. Nopea "kurssi" assemblyn perusteista selkeytti asiaa (Gemini selitti nuo oikean puolen kohdat). Heksakoodi on sinänsä jo tuttua, mutta en sitä kyllä suoriltaan osaisi lukea (vaatisi melkoisesti opiskelua ja muistamista). Koodissa näkyy tosiaan selvästi puts-funktion kutsu (printf koodissa)



## 4. Tekoälyn käyttö

* **Malli:** Gemini (Flash 3.6)
* **Käyttötarkoitus:** Virtuaalikoneen asennusongelmien ratkonta, linux komennot sekä niiden selitys. Markdownin tyylitys apuri. Assembly ja c/c++ opetus/selitys

## 5. Lähteet
GeeksforGeeks, C Hello World Program, luettavissa:  https://www.geeksforgeeks.org/c/c-hello-world-program/  
Stackoverflow, How to strip function names from a C++ production binary in Xcode?, https://stackoverflow.com/questions/76523724/how-to-strip-function-names-from-a-c-production-binary-in-xcode

Linux komennot Geminiltä.
