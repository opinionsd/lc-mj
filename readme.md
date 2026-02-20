# Cel zadania:
Zbudowanie asynchronicznego mechanizmu odbierania zamówień (Webhook) ze sklepu e-commerce i przekazywania ich do systemu ERP. Główne wyzwanie to zapewnienie niezawodności (ERP bywa wolny lub niedostępny) i nieblokowanie po stronie sklepu internetowego.

# Wymagania Główne (MVP)

## 1. Endpoint i Szybka Odpowiedź

Stwórz endpoint POST /webhooks/order-created.
Endpoint musi zwrócić status 202 Accepted natychmiast po udanej walidacji i zleceniu zadania w tło (nie czekamy na odpowiedź z ERP).

## 2. Walidacja Payloadu (TypeScript + Zod / class-validator)

Sprawdź, czy request zawiera wymagane dane (np. eventId, orderId, koszyk). W razie braków zwróć 400 Bad Request.
Logika biznesowa: Jeśli klient podał companyName (klient B2B), wymuś obecność numeru NIP (taxId).

## 3. Idempotencja

Wykorzystaj pole eventId z payloadu, aby zapobiec podwójnemu przetwarzaniu tego samego zdarzenia. Wystarczy prosta weryfikacja w pamięci (np. Set zapisanych ID).
W przypadku wykrycia duplikatu zwróć sukces (np. 200/202), ale zignoruj dalsze procesowanie.

## 4. Asynchroniczna Kolejka i Worker

Zaimplementuj mechanizm kolejkowy (na potrzeby zadania wystarczy rozwiązanie in-memory obrazujące wzorzec, np. EventEmitter, tablica z interwałem lub mock BullMQ).
Stwórz workera, który pobiera zadanie z kolejki, mapuje dane na format dla ERP i symuluje wysyłkę.

## 5. Retry Pattern i Logowanie

Stwórz mock funkcji wysyłającej do ERP, która losowo rzuca błędem (symulacja awarii sieci lub przeciążenia).
Worker musi ponowić próbę wysyłki w przypadku błędu (maksymalnie 3 próby).

Dodaj czytelne logi w kluczowych etapach (np. start przetwarzania, błąd wysyłki, ostateczny sukces/porażka).