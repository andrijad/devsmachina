Važi. Pomozi da kreiramo prvo takvo uputstvo za projekat DevsMachina. Za sada imamo live landing page samo -  https://devsmachina.app/ - sledeći koraci su razvoj same aplikacije i sistema za nju. I to kao MVP u ovoj fazi. U tome ću se oslanjati na instrukcije i rad sa claude.ai u okviru tog projekta.

Prva stvar, pomozi mi da "otkrijem"/navedem koji su sve skills/agents potrebni za ovaj razvoj, uzevši u obzir ove instrukcije.

Note: deo instrukcija su direktno moje i tiču se konteksta i same aplikacije. Drugi deo instrukcija su instrukcije za "skill creator" skill.



-------------------------------------
# DEVSMACHINA — GENERAL REQUIRMENTS OVERVIEW

## 1. Korisnički zahtevi (šta korisnik vidi i koristi)

- Registracija sa sistemom pretplate
- Onboarding wizard za povezivanje store-a i eksternih servisa (WooCommerce, GA4, GSC, ad platforme)
- Dashboard sa pregledom svih aktivnih agenata i ključnih KPI-jeva
- Kreiranje, konfigurisanje i brisanje agenata iz kataloga
- Sistem izveštaja — automatski (nedeljni/mesečni) i on-demand
- Notifikacije za kritične događaje (in-app + email)
- Billing upravljanje — pregled plana, fakture, promena metode plaćanja
- User roles za timski pristup (Owner, Admin, Viewer)

## 2. Arhitekturni zahtevi (šta korisnik ne vidi)

- Multi-tenant arhitektura sa strogom data izolacijom po korisniku — nikada data leakage između tenant-a
- Fazna skalabilnost bez prepisivanja koda (10 → 100 → 500 → 1000+ korisnika)
- Stateless agent worker-i, horizontalno skalabilni
- Job queue sistem sa prioritetnim redovima (kritični alerti > redovni taskovi > izveštaji)
- LLM provider-agnostic sloj — konkretni provajder(i) se biraju naknadno, arhitektura mora podržati multi-provider stack sa fallback logikom
- Adapter/wrapper pattern za sve eksterne API-je — promena u 3rd party servisu = promena adaptera, ne core koda
- Modularni agent sistem — dodavanje novog agenta ne dira postojeću platformu
- Graceful degradation — pad eksternog servisa pauzira agenta i kreira alert, ne ruši sistem

## 3. Razvojni deliverables (interni)

- Security: autentifikacija, autorizacija po ulogama, enkripcija, rate limiting, GDPR compliance
- Monitoring: uptime, strukturirani logovi, agent audit trail (svaka akcija logovana), error tracking
- CI/CD: version control, automated testing, staging environment, zero-downtime deploy
- Backup strategija sa mogućnošću point-in-time recovery
- Dokumentacija: API spec, agent specifikacije, runbook-ovi, korisnička dokumentacija

## 4. Faze razvoja

- Mesec 1–2: core platforma (auth, tenant sistem, store konektor, dashboard, plaćanje)
- Mesec 2–3: prva dva agenta iz postojećih n8n workflow-a (PriceGuard + RankSentinel)
- Mesec 3–4: ConvertOps + IntelReporter
- Mesec 4–5: beta sa 10–20 korisnika, StockSignal, LocaleOps, stabilizacija
- Mesec 6+: optimizacija, horizontal scaling, feature flags, novi alati

## 5. Otvorena pitanja

- Shopify podrška — kada i kako uvesti
- Finalni pricing model i šta svaki tier uključuje
- White-label / partner opcija za agencije
- LLM provider selekcija i dugoročna strategija
- Compliance zahtevi (SOC 2, HIPAA) zavisno od ciljnih industrija
- Scraping legalnost u skali i proxy strategija
- Marketplace za custom agente / plugin ekosistem
- Mobilna app vs responsive web

# FUNKCIONALNOSTI
Ovde opisujemo funkcionalnosti za korisnika u MVP verziji. FUNKCIONALNOSTI treba da budu ovi Monitoring/reporting/on demand agenti/alati:

## 1. Operations Report 

	- Synthesizes GA4, ad spend, and revenue data into a weekly digest with variance analysis and trend alerts.
	- Ali tako da report bude actionable odmah za mkt/web/sales timove. 
	- lokalni prototip je bio kreiran kao n8n workflow koji je čuvao/slao html report.
	- Može biti redovni report ili one time.
	
		
## 2. SEO monitoring
	- na osnovu podataka sa konektovanih izvora (za MVP samo Ga4, eventulno i GSC) - korisnici treba da dobijaju report i jasne instrukcije šta i kako promeniti/unaprediti. 
	- lokalni prototip je bio kreiran kao n8n workflow koji je čuvao/slao html report.
	- Može biti redovni report ili one time.
	
	
## 3. CRO monitoring 
	- slično kao SEO, samo fokus na CRO parametre i unapređenja.
	- lokalni prototip nije bio kreiran ali se može kreirati isti kao SEO monitoring, samo se prompt u AI agentu menja. Eventualno i ulazni podaci.	
	- Može biti redovni report ili one time.
		
## 4. Pricing Intelligence reporting 
	- analizira jedan ili više sajtova konkurencije i izveštava o promenama cena, dodavanju ili uklanjanju proizvoda i aktivnim promocijama/kuponima.
	- Reporting je baziran na trigerimau definisanom intervalu (npr jednom dnevno) se radi provera stanja/promena. Ako je bilo promena šalje report. Ako nije samo čuva info da nije bilo promena, tj poslednje stanje.
	-- lokalni prototip je bio kreiran kao n8n workflow u koji je u toku konfiguracije flowa bila unešena URL adresa za scraping i to je bilo to. Nakon toga workflow bi slao report o promenama na mail i čuvao ih lokalno na serveru i u google drive.
	- Može biti redovni report ili one time. Kad je one time, samo izvuče listu proizvoda sa osnovnim informacijama jer nema istorijske podatke da bi ih poredio.
		
## 5. Inventory Signals reporting 
	- Ovo zavisi od integracija sa 3rd party alatima pa biti oprezan u planiranju.
	- U MVP se fokusirati na povezivanje sa WP/Woo i, eventualno Shopify
	- Alat prati stock level stanje i promene, potražnju i prodaju i šalje alerts/izveštaje.
	- samo redovni reporting (trigger based)
	- nije kreiran lokalni prototip za ovo, ali bi funkcionalnost trebalo da je viable u obimu dovoljnom za MVP.

## 6. Landing Page in-debth auditor 
	- alat koji će omogućiti korisniku da uradi tech SEO, content SEO, Core web vitals i CRO audit, te da kreira report sa jasnim preporukama šta konkretno i kako da popravi na strani.
	- Korisnik može da bira da li će raditi sve audit ili samo neke od dostupnih.
	- lokalni prototip je bio kreiran kao n8n workflow koji je radio on demand. Pokretanje bi otvorilo popup gde se unese URL. Nakon egzekucije n8n je izbacivao ekran sa rezultatima, čuvao iste lokalno na server kao html i davao korisniku opciju "save as (html)". 
		-- on demand only
	
## 7. Localization Coverage Audits 
	- multilingual store coverage, flags untranslated content, and tracks localization gaps across target markets. 
	- Suštinski alat koji će proveriti da li na multijezičnom sajtu (MVP može da se ograniči samo na WP ili samo WP/Woo) postoje prevodi svih strana na dodate jezike, kao i da li je sadržaj tih strana zaista preveden).
	- lokalni prototip je bio kreiran kao Python skripta koja se izvršavala "on demand" i čuvala report kako html/json/csv.
		--Napomena: možda može da se kreira rešenje koje će stalno pratiti da li postoji problem sa prevodima, ali je pitanje koliko je to potrebno, pa zato u MVP ne treba.

# USABILITY/PURPOSE

Ovde opisujem šta i kako aplikacija radi, tačnije kako će korisnici da je koriste i u koje svrhe, te i to šta će korisnici dobiti. Korisnici u dashboard mogu da odaberu agenta koji žele da se pokreće (redovni reporting ili on demand):

## 1. Operations
	- logika je da agent skuplja podatke iz GA4 te da se fokusira na ključne pokazatelje vezane za uspeh poslovanja u nekom intervalu (nedelja, mesec, kvartal(?)). Potrošnja, prodaja, zarada, AOV, ad revenue i ad spending (ako ima u GA4) i slično. 
	- Na osnovu tih podataka kreira detaljan report, ali sa fokusom na akcije/zaključke koje stakeholder može da uradi i to poređane po prioritetima na osnovu Eisenhower Matrix logike.
	- Instrukcije treba da budu jasne i nedvosmislene, npr. "povećajte/smanjite ad budžet za Proizvod X jer razlog Y", "dodajte Kategoriju/Brend/Proizvod X na sajt jer je potražnja za srodnim proizvodima jaka", "ponudite veći popust kupcima iz segmenta X jer im je AOV niži od kupaca iz segmenta Y verovatno jer im je  popust niži".
	- Svaki predlog mora da bude baziran sa jedne strane na sintetičkim pokazateljima/parametrima, a sa druge strane na osnovu dobrih/uspešnih poslovnih praksi.
	- BI/Data-analyst mindset u odlučivanju šta je vredno za akciju.
	
## 2. SEO monitoring/reporting 
	- prati ključne SEO pokazatelje, provera SEO podatke za relevantne strane i predlaže konkretne akcije šta i kako treba da se promeni. 
	- Ista logika kao i za operations reporting - spisak konkretnih aktivnosti koje stakeholder treba da izvede da bi unapredi SEO, prioritetizovano na osnovu Eisenhower Matrix. "META podaci na strani X su loši/fale", "indexiraju se strane koje ne treba", "crawl budget nije optimizovan", "canonical tag je loš"...
	- Apsolutno mandatory je da se primene proverene direktive iz SEO znanja. 
	- SEO direktiva/znanja definisati na osnovu poznatih dobrih praksi za SEO sa pouzdanih izvora, pohranjenog AI SEO skill/agenta, kao i na osnovu dokumenata koje ću da "ubacim" u robota (moji interni vodiči i baza znanja poput "SEO & CONTENT Guide", "website launch/migration tech SEO to-do list" i slično).

## 3. CRO monitoring
	- struktura i logika ista kao za prethodna dva monitor/reporting agenta, ali primenjeno na CRO i ponašanje korisnika na sajtu. 
	- Instrukcije šta popraviti da bi se unapredili CTR, engagement rate, smanjio checkout-exits i slično.
	
## 4. Pricing/competition monitoring
	- prati jedan ili više sajtova konkurencije za promene u pogledu inventory, product liste, cena, promocija...
	- Triger izaziva struktuiranu poruku sa jasno ispisanim informacijama.
	
## 5. Inventory signals
	- Detects low stock risks, seasonal demand shifts, and supplier lead time mismatches before they become stockouts.
	- MVP - bazirano samo na intergaciji sa WP/Woo - Shopify ako nije kompleksno.
	
## 6. Landing Page in-debth auditor
	- unese se url koji treba da se analizira i šta tačno se analizira (CRO; tech SEO, COntent, performance/CWV/PSI pokazatelji...) -> report na definisanu adresu ili u definisanom formatu.
	
## 7.Localization Coverage 
	- provera multijezičkog sajta da li sve strane imaju prevode i ako imaju instance prevoda, te da li je sadržaj na njima zaista preveden (ili je kopija default jezika).
	
# POVEZIVANJE

Ovo se odnosi na to kako će Ai Agenti dobijati inicijalne podatke.

	## 1. Scrapper će analizirati dostupne podatke (drugačije i ne može).
	
	## 2. Auditori i localization coverage alati će biti povezani na sopstveni sajt.
	
	## 3. Inventory - za WP/WOO bi trebalo preko sopstevnog API ili Application Key opcije. (potreban admin). Za ERP/CRM alate - preko API ali vrv ne u MVP. Ovo je tema za razmišljanje u sledećim fazama, a sada samo da se notira.
	
	## 4. GA4 i slični alati - idealno preko OAuth tokena ili što jednostavnije za korisnike. 


# REPORTING

	## 1. Korisnici mogu da biraju na koji način dobijaju izveštaj. 
	## 2. U MVP reporting može da bude samo email (eventualno signal, telegram, slack poruke).
	## 3. U MVP korisnici mogu mapirati kako će koji report da se šalje (npr Inventory ili pricing da ide inventory ili mkt timu, operations report managementu, CRO/tech SEO web/dev timu, itd)
	## 4. Default opcija je slanje na mail.
	## 5. Automatsko rutiranje na osnovu tipa trigera/eventa/konteksta je feature za budućnost.
	## 6. Opcije predlega reporta u samoj aplikaciji uz opcije za - send, save, export, delete. Za MVP možda samo staviti fokus na slanje, bez opcije za pregled i menadžment u samoj app.
	

# HISTORICAL DATA
	## 1. Historical data: Korisnici mogu da vide prethodne reports/audits koje su imali.
	## 2. Mogu ih preuzeti/eksportovati.
	## 3. Posle MVP mogu se omogućiti dodatne opcije
		
	
# OUT OF SCOPE FOR MVP

## 1. META reports 
	- opcija da se više izveštaja objedini u jedan i da se actionable insights ili same konkretne akcije definišu na osnovu holističkog pregleda svih informacija.
	- Ajzenhauer matrica i ovde primenljiva.
	
## 2.integracije sa drugim ERPs i CRM za praćenje inventory. 
	- API calls
	
## 3. integracije sa trećim Mkt/sales alatima
	- semrush, ahrefs, hotjar, klaviyo, mailchimp, aftership (koristan za shipping/logistics analize)...

## 4. redundancy in "external services"
	- odnosi se na pomenutu zavisnost od spoljnih servisa. Nije potrebno samo da se prate njihovi updates, već i da se u osnovnom nivou kreira redundantnost. 
	- Npr ako ćemo koristiti Oauth za autorizaciju prilikom povezivanja sa GA4, potrebno je da imamo (u planu) i način da se kačimo sa "Google Service Account account" ako google odluči da ukine oauth ili obratno. Ovo je samo primer. 
	- Ovo nije deo MVP koda ili koncepta, ali će biti bitno u budućnosti kako ne bi morali da gasimo app jer neki servis više nije moguć.

