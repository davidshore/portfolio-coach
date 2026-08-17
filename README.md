# TypeScript + Next.js – Portfolio Coach

I denna workshop bygger du en liten **Next.js-app med TypeScript** från grunden.

Appen ska fungera som en enkel **Portfolio Coach**.
Användaren skriver in en kort presentation av sig själv och vilka tekniker man vill jobba med. Appen skickar datan till en API-route och får tillbaka ett typat JSON-svar med feedback.

I grunduppgiften använder vi **mockad AI**. Det betyder att svaret genereras av en vanlig TypeScript-funktion. Som extra-utmaning kan du senare koppla på riktig AI, till exempel Google Gemini.

---

## Vad ska appen göra?

Användaren fyller i:

- namn
- målroll, till exempel `Frontend Developer Intern`
- teknikstack, till exempel `React, TypeScript, Next.js`
- kort portfolio-/LIA-pitch

Appen returnerar feedback:

- en score mellan 1 och 100
- styrkor
- förbättringsförslag
- en förbättrad version av pitchen
- konkreta nästa steg

Exempel:

```txt
Input:
Namn: Sara Andersson
Målroll: Frontend Developer Intern
Teknikstack: React, TypeScript, Next.js
Pitch: Jag söker LIA inom frontend och har byggt flera skolprojekt.

Output:
Score: 72/100

Styrkor:
* Tydlig inriktning mot frontend
* Relevant teknikstack

Förbättra:
* Beskriv ett konkret projekt
* Förklara vad du bidrog med

Förbättrad pitch:
Jag söker LIA som Frontend Developer Intern. Jag arbetar med React,
TypeScript och Next.js och vill utvecklas i ett professionellt team.

Nästa steg:
* Lägg till ett konkret projekt
* Länka till GitHub eller en demo
```

---

## Mål

När du är klar ska du ha tränat på att:

- skapa ett Next.js-projekt med TypeScript
- skapa egna typer för input och output
- dela typer mellan frontend och API
- skapa en API-route i Next.js
- validera okänd data på runtime
- göra ett typat `fetch`-anrop
- hantera UI-state med union types
- rendera ett typat JSON-svar i React
- lämna in projektet via GitHub

---

## 1. Skapa projektet

Skapa ett nytt Next.js-projekt:

```bash
npx create-next-app@latest portfolio-coach .
```

Välj ungefär:

```txt
TypeScript: Yes
ESLint: Yes
Tailwind: valfritt
src directory: Yes
App Router: Yes
Turbopack: valfritt
Import alias: Yes
```

Starta projektet:

```bash
cd portfolio-coach
npm run dev
```

---

## 2. Rensa startsidan

Öppna:

```txt
src/app/page.tsx
```

Skapa en enkel startsida med rubrik och en kort beskrivning.

Du behöver inte lägga tid på design i början. Få flödet att fungera först.

---

## 3. Skapa delade typer

Skapa filen:

```txt
src/types/shared.ts
```

Skapa typer för input och output.

Du kan börja ungefär så här:

```ts
export type CoachRequest = {
  name: string;
  targetRole: string;
  techStack: string[];
  pitch: string;
};

export type CoachFeedback = {
  score: number;
  strengths: string[];
  improvements: string[];
  improvedPitch: string;
  nextSteps: string[];
};

export type CoachResponse = {
  request: CoachRequest;
  feedback: CoachFeedback;
  createdAt: string;
};

export type RequestState = "idle" | "loading" | "error" | "ready";
```

Fundera:

- Vilka delar kommer från användaren?
- Vilka delar returneras från API:t?
- Varför är `techStack` en array?

---

## 4. Skapa mockad AI-feedback

Skapa filen:

```txt
src/lib/mockCoach.ts
```

Skapa en funktion som tar emot `CoachRequest` och returnerar `CoachResponse`.

Du behöver inte göra den smart. Den kan returnera ganska statiska svar, men försök använda något från användarens input.

Exempel på idé:

```ts
export function createMockFeedback(request: CoachRequest): CoachResponse {
  // returnera ett objekt som matchar CoachResponse
}
```

Krav:

- funktionen ska ta emot rätt typ
- funktionen ska returnera rätt typ
- svaret ska innehålla minst 2 styrkor
- svaret ska innehålla minst 2 förbättringsförslag
- svaret ska innehålla minst 2 nästa steg

---

## 5. Validera input på runtime

TypeScript kontrollerar din egen kod, men data från ett formulär/API är okänd när programmet körs.

Skapa filen:

```txt
src/lib/validation.ts
```

Skapa en type guard:

```ts
export function isCoachRequest(value: unknown): value is CoachRequest {
  // kontrollera att value är ett objekt
  // kontrollera att name, targetRole och pitch är strings
  // kontrollera att techStack är en array med strings
}
```

Tips:

- börja med att kontrollera `typeof value === "object"`
- kom ihåg att `null` också räknas som object i JavaScript
- använd `Array.isArray`
- använd `.every(...)` för att kontrollera array-innehåll

---

## 6. Skapa API-route

Skapa filen:

```txt
src/app/api/coach/route.ts
```

API-routen ska:

1. läsa JSON från requesten
2. behandla datan som `unknown`
3. validera datan med din type guard
4. returnera status `400` om datan är fel
5. returnera mockad feedback om datan är rätt

Tips på struktur:

```ts
export async function POST(req: Request) {
  const body: unknown = await req.json();

  // validera body

  // skapa feedback

  // returnera JSON
}
```

Testa gärna API:t innan du bygger hela frontend.

---

## 7. Skapa en typad klientfunktion

Skapa filen:

```txt
src/lib/client.ts
```

Skapa en funktion:

```ts
export async function getCoachFeedback(
  request: CoachRequest,
): Promise<CoachResponse> {
  // gör fetch mot /api/coach
}
```

Krav:

- funktionen ska ta emot `CoachRequest`
- funktionen ska returnera `Promise<CoachResponse>`
- funktionen ska kasta ett error om API:t svarar med fel

Fundera:

- Varför blir returtypen en `Promise`?
- Varför är `fetch` asynkront?
- Vad händer om API:t svarar med status 400?

---

## 8. Skapa formuläret

Skapa filen:

```txt
src/components/PortfolioCoachForm.tsx
```

Komponenten ska:

- vara en client component
- ha state för formulärfälten
- ha state för request-läge: `idle`, `loading`, `error`, `ready`
- ha state för resultatet
- skicka datan till API:t när formuläret submitas

Fält:

- namn
- målroll
- teknikstack som kommaseparerad text
- pitch som textarea

När du skickar datan behöver du göra om teknikstacken från text till array.

Exempel:

```ts
const techStack = techStackInput
  .split(",")
  .map((item) => item.trim())
  .filter(Boolean);
```

---

## 9. Rendera resultatet

När API:t har svarat ska du visa:

- score
- styrkor
- förbättringsförslag
- förbättrad pitch
- nästa steg

Du ska också visa olika innehåll beroende på state:

```txt
idle    → visa en kort instruktion
loading → visa att feedback hämtas
error   → visa felmeddelande
ready   → visa feedback
```

---

## 10. Använd komponenten på startsidan

Importera din komponent i:

```txt
src/app/page.tsx
```

och rendera den på sidan.

---

## 11. Kontrollera projektet

Kör:

```bash
npm run lint
npm run build
```

Fixa fel innan du lämnar in.

---

## 12. Lägg upp på GitHub

Skapa ett repo på GitHub och pusha projektet.

Lämna in GitHub-länken i Codington.

---

# Krav för G

För godkänt ska appen ha:

- ett Next.js-projekt skapat från grunden
- delade typer i `src/types/shared.ts`
- mockad AI-feedback i en separat funktion
- runtime-validering av input
- API-route: `src/app/api/coach/route.ts`
- typad klientfunktion som gör `fetch`
- formulär med minst fyra fält
- UI-state: `idle`, `loading`, `error`, `ready`
- renderad feedback från API:t
- projektet inlämnat via GitHub

---

# Extra-utmaningar för högre nivå

Välj minst två.

## 1. Använd Zod

Byt ut din egen type guard mot Zod-validering.

## 2. Koppla på riktig AI

Byt ut mock-funktionen mot ett riktigt AI-anrop.

Viktigt:

- lägg API-nyckeln i `.env.local`
- pusha inte `.env.local`
- returnera fortfarande ett strukturerat JSON-svar

## 3. Bättre feedback

Gör så att feedbacken ändras mer beroende på användarens input.

Exempel:

- om pitchen är väldigt kort, föreslå mer konkreta projekt
- om teknikstacken saknar TypeScript, föreslå att lägga till typade exempel
- om målrollen är frontend, ge frontend-specifika råd

## 4. Mobilanpassa sidan

Gör sidan tydligare och mer användbar på mobil.

## 5. Lägg till tester

Testa till exempel:

- valideringsfunktionen
- mock-funktionen
- att komponenten visar loading/error/ready

## 6. Spara historik

Låt användaren se tidigare feedback i samma session.

---

# Reflektion

Svara kort i README eller i Codington-inlämningen:

Varför behöver data som kommer till API-routen valideras, även om vi använder
TypeScript och typen `CoachRequest`?

---

# Success ✅

När du är klar har du byggt ett modernt TypeScript-flöde i Next.js:

```txt
Formulär → typed fetch → API-route → runtime-validering → mockad AI → typat JSON-svar → UI-state → renderad feedback
```

Det här är samma grundmönster som används i många riktiga webbappar.
