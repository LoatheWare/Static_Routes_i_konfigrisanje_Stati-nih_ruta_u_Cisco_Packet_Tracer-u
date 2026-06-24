# Statičke rute i konfigurisanje statičkih ruta u Cisco Packet Tracer-u
## Šta je ovo?
Ovo je vežba za administratore računarskih mreža koju sam radio tokom mog školovanja. Nalazi se jedan .pkt fajl koji se preuzme i to će Vam biti početno stanje. Zadak Vam je dat, kao i uputstvo za njegovo rešavanje. Ukoliko Vam nešto nije jasno tokom rešavanja ovih zadataka obratite mi se direktnom porukom na aplikaciji LinkedIn, www.linkedin.com/in/nikola-karanović-397185390

## Pitanja i odgovori — Statičke rute (Static Routes)

### 1. Šta je statička ruta?
**Statička ruta** je ruta koju **administrator ručno** unosi u routing tabelu rutera, kako bi definisao putanju do određene udaljene mreže. Za razliku od dinamičkih protokola rutiranja (OSPF, EIGRP...), statička ruta se **ne menja automatski** prilikom promena u mreži — ako link otkaže, a ne postoji alternativna ruta, ruter neće sam pronaći drugi put, već je potrebna intervencija administratora (osim u slučaju floating static route-a, o čemu kasnije).

---

### 2. Koje su prednosti statičkih ruta?
Statičke rute nam omogućavaju:
1. **Veću sigurnost i kontrolu** — administrator tačno zna i određuje koji put saobraćaj koristi, bez mogućnosti da neki neovlašćeni ruter "ubaci" lažnu rutu.
2. **Manje opterećenje rutera** — nema procesiranja algoritama rutiranja (npr. SPF kod OSPF-a), niti razmene update poruka, što štedi CPU i propusni opseg.
3. **Predvidljivost** — putanja saobraćaja je fiksna i ne menja se dinamički, što je korisno za testiranje, sigurnosne politike ili manje, stabilne mreže.
4. **Jednostavnost u malim mrežama** — u mrežama sa malim brojem rutera, statičke rute su brže i lakše za konfigurisanje od podizanja čitavog protokola dinamičkog rutiranja.

---

### 3. Koji su nedostaci statičkih ruta?
- **Ne skaliraju se dobro** — u velikim mrežama sa puno podmreža, ručno unošenje i održavanje svake rute postaje nepraktično.
- **Nema automatske adaptacije** — ukoliko se topologija mreže promeni (otkaže link), statička ruta **ne reaguje automatski**, sem ako postoji floating static ruta kao backup.
- **Veća mogućnost ljudske greške** — pošto se sve unosi ručno, lako je pogrešiti IP adresu, masku ili next-hop.

---

### 4. Koje vrste statičkih ruta postoje (prema next-hop-u)?
Statičke rute se mogu konfigurisati na tri načina, u zavisnosti od toga šta se navodi kao "sledeći korak":
- **Next-hop statička ruta** — navodi se IP adresa sledećeg rutera (next-hop adresa).
- **Directly connected (exit interface) statička ruta** — navodi se izlazni interfejs rutera (npr. Serial0/0/0), umesto IP adrese.
- **Fully specified statička ruta** — navodi se **i** izlazni interfejs **i** next-hop IP adresa istovremeno (preporučeno na multiaccess mrežama, npr. Ethernet, da bi se izbegao dodatni ARP overhead).

---

### 5. Koja je osnovna sintaksa komande za konfigurisanje statičke rute?
Opšti format komande:

"Router(config)# ip route [mreža] [maska] [next-hop-adresa ili izlazni-interfejs] [AD]"

Primer sa next-hop adresom:

"Router(config)# ip route 192.168.20.0 255.255.255.0 10.0.0.2"

Primer sa izlaznim interfejsom:

"Router(config)# ip route 192.168.20.0 255.255.255.0 Serial0/0/0"

Primer fully specified rute:

"Router(config)# ip route 192.168.20.0 255.255.255.0 Serial0/0/0 10.0.0.2"

---

### 6. Kako se konfiguriše statička ruta za IPv6?
Princip je identičan kao kod IPv4, samo se koristi komanda `ipv6 route`:

"Router(config)# ipv6 route 2001:DB8:ACAD:2::/64 2001:DB8:ACAD:1::2"

---

### 7. Šta je Administrativna distanca (AD) i kako utiče na statičke rute?
**Administrativna distanca (AD)** je vrednost koja predstavlja **pouzdanost izvora rute** — što je AD **manja**, ruta se smatra **pouzdanijom** i ima veći prioritet u routing tabeli. Statička ruta ima **default AD = 1**, što je vrlo nizak broj (pouzdano), pa će po pravilu **uvek imati prednost** nad rutama naučenim dinamičkim protokolima (npr. OSPF AD=110, EIGRP AD=90), osim ako je AD ručno promenjena.

AD se ručno podešava na kraju komande, kao dodatni parametar:

"Router(config)# ip route 192.168.20.0 255.255.255.0 10.0.0.2 5"

---

### 8. Šta je Summary (Sumarna) statička ruta?
**Sumarna statička ruta** je jedna statička ruta koja **objedinjuje (sumira) više pojedinačnih mreža** u jedan veći opseg, korišćenjem odgovarajuće (kraće) maske. Umesto da se unosi posebna ruta za svaku od nekoliko susednih podmreža, unosi se jedna ruta koja ih sve obuhvata, čime se **smanjuje veličina routing tabele** i pojednostavljuje konfiguracija.

Primer — umesto dve rute za 192.168.10.0/24 i 192.168.11.0/24, koristi se jedna sumarna ruta:

"Router(config)# ip route 192.168.8.0 255.255.248.0 10.0.0.2"

---

### 9. Šta je floating static route?
**Floating static route** je statička ruta kojoj je **ručno podignuta administrativna distanca** iznad vrednosti primarne rute (statičke ili dinamičke), tako da se **ne koristi normalno**, već služi kao **backup ruta**. Ako primarna ruta (npr. ona naučena preko OSPF-a ili primarna statička ruta) ispadne iz routing tabele (npr. zbog pada linka), floating ruta automatski preuzima ulogu glavne rute.

Primer floating statičke rute (AD podignuta na 100, veće od OSPF AD=110... pa treba paziti da bude veće i od te vrednosti ako treba da bude "poslednja linija odbrane"):

"Router(config)# ip route 192.168.20.0 255.255.255.0 10.0.0.2 100"

---

### 10. Šta je Default Route i kako se uklapa u statičke rute?
**Default route** (podrazumevana ruta) je specifičan tip statičke rute koja se koristi za **sav saobraćaj koji ne odgovara nijednoj drugoj, specifičnijoj ruti** u tabeli. Konfiguriše se sa mrežom i maskom 0.0.0.0 0.0.0.0:

"Router(config)# ip route 0.0.0.0 0.0.0.0 10.0.0.2"

Najčešće se koristi na **stub mrežama** (mrežama sa samo jednim izlazom), kao npr. ka internetu preko ISP-a.

---

### 11. Koje su komande za proveru statičkih ruta na ruteru?
Najvažnije komande za verifikaciju:

"Router# show ip route"

"Router# show ip route static"

"Router# show running-config | include ip route"

- `show ip route` — prikazuje kompletnu routing tabelu; statičke rute su obeležene oznakom **S** (ili **S\*** za statičku default rutu).
- `show ip route static` — filtrira prikaz samo na statičke rute.
- `show running-config | include ip route` — prikazuje sve konfigurisane `ip route` komande direktno iz konfiguracije rutera.

---

### 12. Šta se dešava ako je next-hop adresa u statičkoj ruti nedostupna?
Ukoliko next-hop adresa navedena u statičkoj ruti **nije dostupna** (npr. fizički link je pao, ili ARP ne može da se izvrši), ta statička ruta se **uklanja iz aktivne routing tabele** sve dok ponovo ne postane dostupna (ruter periodično provjerava). Ako u tom periodu postoji **floating static ruta** ili druga alternativna putanja, ona preuzima ulogu prosleđivanja saobraćaja.

---
