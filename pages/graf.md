# eKoruny Tracker - Kompletní dokumentace pro AI

## Přehled projektu

**eKoruny Tracker** je inteligentní webová aplikace pro sledování pokroku k finančnímu cíli v eKorunách. Aplikace je navržena jako statický HTML soubor s JavaScriptem, který funguje offline a ukládá data do localStorage.

## Základní parametry

- **Období:** 11. června 2025 - 30. září 2025
- **Celkový cíl:** 27 842 eKorun
- **Pracovní dny:** Pouze pondělí-pátek (víkendy vyloučeny)
- **Ukládání dat:** localStorage (klíč: `eKorunyProgressData_v7_workdays`)

## Funkční specifikace

### 1. Základní funkcionalita

#### Generování dat
```javascript
// Aplikace generuje data pouze pro pracovní dny
function isWorkDay(date) {
  const day = date.getDay()
  return day !== 0 && day !== 6 // Není neděle ani sobota
}
```

#### Plánování
- **Denní cíl:** `celkový_cíl / (počet_pracovních_dní - 1)`
- **Kumulativní plán:** Lineární nárůst od 0 do celkového cíle
- **Kritická mez:** 80% hodnoty plánu pro každý den

### 2. Inteligentní "Upravený plán" (Záchranná strategie)

#### Logika aktivace
```javascript
// Aktivuje se pouze při skluzu
if (aktuální_realita < aktuální_plán && aktuální_plán > 0) {
  adjustmentNeeded = true
  
  // Výpočet nového denního cíle
  const zbývající_cíl = celkový_cíl - aktuální_realita
  const zbývající_dny = počet_zbývajících_pracovních_dní
  const nový_denní_cíl = zbývající_cíl / zbývající_dny
}
```

#### Zobrazení v tabulce
- **Při skluzu:** Červené pozadí + nový kumulativní plán + denní cíl
- **Při úspěchu:** Zelené pozadí + "✓ Jste v pořádku!"
- **Minulé dny:** "–"

### 3. Vizualizace

#### Graf (Chart.js)
```javascript
datasets: [
  { 
    label: 'REALITA', 
    data: kumulativní_realita,
    borderColor: '#007bff', // Modrá
    borderWidth: 3
  },
  {
    label: 'PLÁN',
    data: kumulativní_plán,
    borderColor: '#6c757d', // Šedá
    borderDash: [5, 5] // Přerušovaná
  },
  { 
    label: 'KRITICKÁ MEZ', 
    data: kritická_mez,
    borderColor: '#dc3545', // Červená
    borderDash: [2, 2]
  }
]
```

#### Tabulka
- **Dnešní den:** Žluté pozadí (`is-today` class)
- **Vstupní pole:** `type="number"`, auto-save do localStorage
- **Kumulativní hodnoty:** Automatický přepočet při změně

### 4. Ukládání a načítání dat

#### Struktura dat v localStorage
```javascript
[
  {
    dateStr: "2025-06-11T00:00:00.000Z",
    dailyInput: 150.50
  },
  // ... další dny
]
```

#### Auto-save
- Každá změna v input poli automaticky ukládá
- Při načtení stránky se data obnovují
- Validace: kontrola shody dat s aktuální strukturou

### 5. Import a Export funkcionalita

#### CSV Import
```javascript
function importCSV(file) {
  // Očekávaný formát CSV:
  // Datum, Den v týdnu, Získané eKč za den, eKč celkem, Původní plán, Kritická mez, Upravený plán
  
  // Automatická detekce a párování dat podle data
  // Podporuje formát data: DD.MM.YYYY
  // Přepíše existující dailyInput hodnoty
}
```

#### CSV Export
- Záhlaví v češtině
- Všechny sloupce včetně upraveného plánu
- Název souboru: `eKoruny_tracker_YYYY-MM-DD.csv`

#### Excel Export (XLSX) - Vylepšený
- **List "Data":** Kompletní tabulka s metadaty
- **List "Graf":** Připravená data pro vytvoření grafu v Excelu
  - Sloupce: Datum, Realita, Plán, Kritická mez
  - Optimalizované pro Excel charty
- Použití SheetJS knihovny z CDN
- Nastavené šířky sloupců pro lepší čitelnost
- Název souboru: `eKoruny_tracker_YYYY-MM-DD.xlsx`

## Technické detaily

### Závislosti (CDN)
```html
<script src="https://cdn.jsdelivr.net/npm/chart.js"></script>
<script src="https://cdnjs.cloudflare.com/ajax/libs/xlsx/0.18.5/xlsx.full.min.js"></script>
```

### Klíčové CSS třídy
```css
.is-today { background-color: #fff3cd; } /* Dnešní den */
.adjusted-plan-cell.is-behind { color: #d9534f; background-color: #f8d7da; } /* Skluz */
.adjusted-plan-cell.is-ahead { color: #28a745; background-color: #d4edda; } /* Úspěch */
.download-button { background-color: #28a745; } /* Export tlačítka */
.import-button { background-color: #17a2b8; } /* Import tlačítko */
.file-input { display: none; } /* Skrytý file input */
```

### Struktura dat
```javascript
chartData = [
  {
    date: Date object,
    plan: number,           // Kumulativní plán
    critical: number,       // 80% plánu
    dailyInput: number,     // Uživatelský vstup
    reality: number,        // Kumulativní realita
    workDayIndex: number    // Index pracovního dne
  }
]
```

## Běžné úpravy a rozšíření

### Změna období
```javascript
const startDate = new Date('YYYY-MM-DDTHH:mm:ss')
const endDate = new Date('YYYY-MM-DDTHH:mm:ss')
```

### Změna cíle
```javascript
const totalGoal = 27842 // Nová hodnota
```

### Změna localStorage klíče (pro reset dat)
```javascript
const LOCAL_STORAGE_KEY = 'eKorunyProgressData_v8_newversion'
```

### Přidání nového sloupce do tabulky
1. Upravit HTML struktur v `row.innerHTML`
2. Přidat CSS styling
3. Aktualizovat export funkce (CSV i Excel)
4. Přidat logiku do `updateAll()` funkce

## Nejčastější problémy a řešení

### 1. Dnešní den se nezvýrazňuje
**Příčina:** Špatné nastavení roku v `startDate`/`endDate`
**Řešení:** Zkontrolovat, že rok odpovídá aktuálnímu roku

### 2. Upravený plán se neaktivuje
**Příčina:** Problém v logice porovnání nebo detekce skluzu
**Řešení:** 
```javascript
console.log('Debug:', {
  realita: currentReality,
  plán: currentPlan,
  pozadu: currentReality < currentPlan
})
```

### 3. Data se neukládají
**Příčina:** localStorage blokován nebo chyba v JSON serializaci
**Řešení:** Zkontrolovat konzoli prohlížeče a localStorage podporu

### 4. Graf se nezobrazuje
**Příčina:** Chart.js se nenačetl z CDN nebo canvas element neexistuje
**Řešení:** Zkontrolovat síťové připojení a DOM strukturu

### 5. CSV import nefunguje
**Příčina:** Nesprávný formát CSV nebo kódování
**Řešení:** 
- Zkontrolovat formát: `Datum,Den,Získané eKč,...`
- Zkontrolovat kódování souboru (UTF-8)
- Datum ve formátu DD.MM.YYYY

### 6. Excel export nemá druhý list
**Příčina:** Chyba v XLSX knihovně nebo vytváření druhého listu
**Řešení:** Zkontrolovat konzoli a dostupnost XLSX.utils funkcí

## Rozvojové směry

### Možná vylepšení
1. **Týdenní cíle:** Přidání mezicílů po týdnech
2. **Poznámky:** Možnost přidávat poznámky k jednotlivým dnům
3. **Více cílů:** Sledování více než jednoho cíle současně
4. **Pokročilé statistiky:** Průměrné denní hodnoty, trendy, predikce
5. **Excel import:** Rozšíření importu o XLSX soubory
6. **Grafické export:** Export grafu jako obrázek (PNG/SVG)
7. **Batch import:** Import více souborů najednou
8. **Offline PWA:** Přeměna na Progressive Web App
9. **Cloud sync:** Synchronizace dat mezi zařízeními

### Architektura pro rozšíření
```javascript
// Modulární struktura pro budoucí rozšíření
const EkorunyTracker = {
  config: { /* nastavení */ },
  data: { /* datová vrstva */ },
  ui: { /* UI funkce */ },
  chart: { /* graf funkcionalita */ },
  storage: { /* ukládání/načítání */ },
  export: { /* export funkce */ }
}
```

## Pokyny pro AI asistenty

### Co zachovat
- Statickou povahu aplikace (jeden HTML soubor)
- Českou lokalizaci
- Základní barevné schéma a UX
- Logiku upraveného plánu
- localStorage ukládání

### Co lze upravovat
- Styling a layout
- Přídávání nových funkcí
- Optimalizaci kódu
- Rozšíření export možností
- Přidání validací

### Testovací scénáře
1. **Normální používání:** Zadávání dat den po dni
2. **Skluz:** Záměrně nízké hodnoty -> kontrola aktivace upraveného plánu
3. **Úspěch:** Vysoké hodnoty -> kontrola zobrazení "v pořádku"
4. **CSV Export:** Test exportu a kontrola formátu
5. **Excel Export:** Test exportu se dvěma listy (Data + Graf)
6. **CSV Import:** Test importu z externího CSV souboru
7. **Persistence:** Obnovení stránky -> kontrola zachování dat
8. **Import/Export cyklus:** Export -> vymazání dat -> import -> kontrola integrity

### Debug příkazy
```javascript
// Zobrazit aktuální stav
console.log('ChartData:', chartData)
console.log('LocalStorage:', localStorage.getItem(LOCAL_STORAGE_KEY))

// Vymazat data pro test
localStorage.removeItem(LOCAL_STORAGE_KEY)
location.reload()

// Simulovat data
chartData.forEach((day, i) => {
  if (i < 5) day.dailyInput = 100 + Math.random() * 100
})
updateAll()
```

---

*Tento dokument obsahuje kompletní technickou dokumentaci pro práci s eKoruny Tracker aplikací. Při jakýchkoli úpravách je důležité zachovat základní funkcionalita a uživatelskou přívětivost nástroje.*