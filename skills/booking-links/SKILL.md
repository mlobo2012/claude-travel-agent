---
name: booking-links
description: "Canonical source of truth for constructing deep booking URLs across flights, trains, ferries, and accommodation. Ensures every search result links to the specific option with dates, passengers, and route pre-filled — never a generic homepage. Automatically active — not user-invocable."
user-invocable: false
---

# Booking Links — Deep Link Construction Reference

This skill is the **canonical source of truth** for how the AI Heroes Travel Agent constructs booking URLs. Every search skill (flights, trains, ferries, accommodation) references this skill. The rules here override any conflicting guidance elsewhere.

## ABSOLUTE RULES

1. **Never link to a generic homepage.** Every booking link MUST include route, dates, and passengers pre-filled in the URL parameters. A link to `ryanair.com` is WRONG. A link to `ryanair.com/gb/en/trip/flights/select?adults=2&dateOut=2026-05-14&originIata=STN&destinationIata=CTA` is CORRECT.
2. **Link text MUST name the specific option.** Write `[Book Ryanair STN→CTA 14 May](URL)` — NOT `[Search on Ryanair](URL)` or `[Book this flight](URL)`. Include operator/airline, route, and date or time. Flight number too if known.
3. **Format as markdown links.** Never use angle brackets, plain text URLs, or `<a>` tags.
4. **Include children in every URL** that supports child parameters when children are in the party.
5. **Include loyalty/membership numbers** in URLs where the platform supports it.
6. **Use the operator's direct site first** when a deep link template exists. Use aggregators (Google Flights, Omio, Ferryhopper) as fallback only.
7. **One-way trips**: omit return date parameters entirely — do not pass empty or null values.

---

## FLIGHT DEEP LINKS

### Airline Direct Links (preferred)

| Airline | URL Template |
|---------|-------------|
| **Ryanair** | `https://www.ryanair.com/gb/en/trip/flights/select?adults={adults}&teens=0&children={children}&infants={infants}&dateOut={YYYY-MM-DD}&dateIn={YYYY-MM-DD}&isConnectedFlight=false&discount=0&isReturn=true&promoCode=&originIata={ORIGIN}&destinationIata={DEST}` |
| **easyJet** | `https://www.easyjet.com/en/booking/select-flight?pid=www.easyjet.com&origin={ORIGIN}&destination={DEST}&oday={DD}&omonth={MM}&oyear={YYYY}&rday={DD}&rmonth={MM}&ryear={YYYY}&adults={adults}&children={children}&infants={infants}` |
| **British Airways** | `https://www.britishairways.com/travel/book/public/en_gb?from={ORIGIN}&to={DEST}&depDate={YYYYMMDD}&retDate={YYYYMMDD}&cabin=M&adultCount={adults}&youngAdultCount=0&childCount={children}&infantCount={infants}&eTicket=e` |
| **Wizz Air** | `https://wizzair.com/en-gb/booking/select-flight/{ORIGIN}/{DEST}/{YYYY-MM-DD}/{YYYY-MM-DD}/{adults}/{children}/{infants}/0` |
| **Vueling** | `https://www.vueling.com/en/booking/select?origin={ORIGIN}&destination={DEST}&outboundDate={YYYY-MM-DD}&inboundDate={YYYY-MM-DD}&adults={adults}` |
| **Lufthansa** | `https://www.lufthansa.com/gb/en/flight-search?origin={ORIGIN}&destination={DEST}&outDate={YYYY-MM-DD}&inDate={YYYY-MM-DD}&paxCombination={adults}-0-{children}-{infants}-0` |
| **KLM** | `https://www.klm.com/search/result?pax={adults}&cabin=ECONOMY&lang=en&origins={ORIGIN}&destinations={DEST}&outDate={YYYY-MM-DD}&inDate={YYYY-MM-DD}` |
| **Air France** | `https://www.airfrance.co.uk/search/offer?pax={adults}ADT&cabinClass=ECONOMY&activeConnection=0&connections=A{ORIGIN}-A{DEST}-{YYYYMMDD}~A{DEST}-A{ORIGIN}-{YYYYMMDD}` |
| **Iberia** | `https://www.iberia.com/gb/flights/?market=GB&language=en&appliesOMB=N&TRIP_TYPE=2&BEGIN_TRIP_1={DD}/{MM}/{YYYY}&BEGIN_TRIP_2={DD}/{MM}/{YYYY}&ORIGIN_1={ORIGIN}&DESTINATION_1={DEST}&ADT={adults}&CHD={children}&INF={infants}` |
| **TAP Portugal** | `https://www.flytap.com/en-gb/booking?type=R&from={ORIGIN}&to={DEST}&out={YYYY-MM-DD}&in={YYYY-MM-DD}&adt={adults}&chd={children}&inf={infants}` |
| **Aer Lingus** | `https://www.aerlingus.com/booking/select-flights?adult={adults}&child={children}&infant={infants}&trip=RETURN&iataFrom={ORIGIN}&iataTo={DEST}&departureDate={YYYY-MM-DD}&returnDate={YYYY-MM-DD}` |
| **Norwegian** | `https://www.norwegian.com/en/booking/flight-offers/?D_City={ORIGIN}&A_City={DEST}&D_Day={DD}&D_Month={YYYYMM}&R_Day={DD}&R_Month={YYYYMM}&AdultCount={adults}&ChildCount={children}&InfantCount={infants}` |
| **SAS** | `https://www.flysas.com/en/book/flights?origin={ORIGIN}&destination={DEST}&outDate={YYYY-MM-DD}&inDate={YYYY-MM-DD}&adt={adults}&chd={children}&inf={infants}` |
| **Turkish Airlines** | `https://www.turkishairlines.com/en-gb/flights/?origin={ORIGIN}&destination={DEST}&departureDate={YYYYMMDD}&returnDate={YYYYMMDD}&adult={adults}&child={children}&infant={infants}` |
| **Emirates** | `https://www.emirates.com/gb/english/book/?from={ORIGIN}&to={DEST}&class=Economy&adult={adults}&child={children}&infant={infants}&departure={YYYYMMDD}&return={YYYYMMDD}` |
| **Qatar Airways** | `https://www.qatarairways.com/en-gb/booking/book-flight.html?from={ORIGIN}&to={DEST}&departing={YYYY-MM-DD}&returning={YYYY-MM-DD}&adults={adults}&children={children}&infants={infants}&class=Economy` |

### Loyalty Number in Flight URLs

| Airline | How to include |
|---------|---------------|
| **British Airways** | Append `&eId={MEMBERSHIP_NUMBER}` to the BA URL |
| **Lufthansa** | Append `&MESSION={MEMBERSHIP_NUMBER}` |
| **Others** | Most airlines require login — mention: "Your {programme} number {XXXX} — log in before booking to earn points" |

### Fallback: Google Flights Deep Link

When no airline-specific template exists, or for any airline not listed above:

```
https://www.google.com/travel/flights?q=Flights+from+{ORIGIN}+to+{DEST}+on+{YYYY-MM-DD}+returning+{YYYY-MM-DD}&curr={CURRENCY}&px={TOTAL_PAX}
```

- Replace spaces with `+`
- Include currency from user profile
- Include total passenger count
- For one-way, omit `+returning+{date}`

**Link text example:** `[Book BA548 LHR→FCO 14 May](URL)` — always include airline, flight number (if known), route, and date.

---

## TRAIN DEEP LINKS

| Operator/Platform | URL Template |
|-------------------|-------------|
| **Eurostar** | `https://www.eurostar.com/uk-en/train/booking/options?origin={STATION_CODE}&destination={STATION_CODE}&outbound-date={YYYY-MM-DD}&adult={adults}&youth={youth_4_11}&child={child_0_3}` |
| **Trainline (UK)** | `https://www.thetrainline.com/book/results?origin={STATION_ID}&destination={STATION_ID}&outwardDate={YYYY-MM-DD}T{HH}%3A00%3A00&outwardDateType=departAfter&journeySearchType=single&passengers%5B%5D={DOB}%7Cadult&temporalDirection=departing` |
| **Trainline (EU)** | `https://www.trainline.eu/search/{ORIGIN_CITY}/{DEST_CITY}/{YYYY-MM-DD}` |
| **Deutsche Bahn** | `https://int.bahn.de/en/buchung/fahrplanauskunft?reise.von={ORIGIN}&reise.nach={DEST}&reise.datum={YYYY-MM-DD}&reise.zeit={HH}%3A{MM}&reise.zeitpunktart=ABFAHRT` |
| **SNCF Connect** | `https://www.sncf-connect.com/en-en/train/results?departure={ORIGIN}&arrival={DEST}&date={YYYY-MM-DD}&passengers={N}adult` |
| **Omio** | `https://www.omio.com/search-frontend/results/{ORIGIN}/{DEST}?date={YYYY-MM-DD}&passengers={N}` |
| **Amtrak** | `https://www.amtrak.com/tickets/departure.html?origin={CODE}&destination={CODE}&departing={MM/DD/YYYY}&adults={N}&children={N}` |
| **Italo (Italy)** | `https://www.italotreno.it/en/offers?origin={ORIGIN}&destination={DEST}&outbound_date={YYYY-MM-DD}&adults={N}` |
| **Renfe (Spain)** | `https://www.renfe.com/es/en/cercanias?origen={ORIGIN}&destino={DEST}&fecha={DD/MM/YYYY}` |

### Eurostar Station Codes

| Station | Code |
|---------|------|
| London St Pancras | `7015400` |
| Paris Gare du Nord | `8727100` |
| Brussels-Midi | `8814001` |
| Amsterdam Centraal | `8400058` |
| Rotterdam Centraal | `8400530` |
| Lille Europe | `8728600` |

### Children in Train URLs

| Platform | How to add children |
|----------|-------------------|
| **Eurostar** | `&child={N_0_3}&youth={N_4_11}` |
| **Trainline** | Add `&passengers%5B%5D={DOB}%7Cchild` per child |
| **Deutsche Bahn** | `&reise.kinder={N}` |
| **SNCF** | `&passengers={N}adult,{N}child` |
| **Omio** | `&passengers={N}adults,{N}children` |
| **Amtrak** | `&children={N}` |

### Train Loyalty Notes

| Programme | How to handle |
|-----------|--------------|
| **Club Eurostar** | URL does not support pre-fill — say: "Your Club Eurostar number {XXXX} — log in before booking to earn points" |
| **BahnCard** | URL does not support pre-fill — say: "Your BahnCard {type} — enter at checkout for discounts" |
| **UK Railcards** | URL does not support pre-fill — say: "Apply your {Railcard name} at checkout for 1/3 off" |

**Link text example:** `[Book Eurostar 9014 London→Paris 08:01](URL)` — always include operator, service number (if known), route, and departure time.

---

## FERRY DEEP LINKS

| Operator/Platform | URL Template |
|-------------------|-------------|
| **Ferryhopper** | `https://www.ferryhopper.com/en/ferry-routes/{ORIGIN}-{DEST}?departureDate={YYYY-MM-DD}&passengers={N}` |
| **Direct Ferries** | `https://www.directferries.com/routes/{ORIGIN}_{DEST}.htm?departureDate={YYYY-MM-DD}&adults={N}` |
| **DFDS** | `https://www.dfds.com/en/passenger-ferries/{ORIGIN}-{DEST}?outbound={YYYY-MM-DD}&adults={N}` |
| **Stena Line** | `https://www.stenaline.co.uk/routes/{ORIGIN}-{DEST}?date={YYYY-MM-DD}&adults={N}` |
| **P&O Ferries** | `https://www.poferries.com/en/routes/{ORIGIN}-{DEST}?departureDate={YYYY-MM-DD}&adults={N}` |
| **Brittany Ferries** | `https://www.brittany-ferries.co.uk/ferry-routes/{ORIGIN}-{DEST}?departureDate={YYYY-MM-DD}&adults={N}` |
| **Blue Star Ferries** | `https://www.bluestarferries.com/en/booking?from={ORIGIN}&to={DEST}&date={YYYY-MM-DD}&passengers={N}` |
| **Viking Line** | `https://www.vikingline.com/find-a-cruise-or-ferry/?route={ORIGIN}-{DEST}&date={YYYY-MM-DD}&adults={N}` |
| **Jadrolinija** | `https://www.jadrolinija.hr/en/ferry-croatia?from={ORIGIN}&to={DEST}&date={YYYY-MM-DD}&adults={N}` |
| **Irish Ferries** | `https://www.irishferries.com/ie-en/routes/{ORIGIN}-{DEST}/?sailing_date={YYYY-MM-DD}&adults={N}` |
| **Color Line** | `https://www.colorline.com/ferries/{ORIGIN}-{DEST}?departureDate={YYYY-MM-DD}&adults={N}` |

### Children in Ferry URLs

| Platform | How to add children |
|----------|-------------------|
| **Ferryhopper** | `&children={N}` |
| **Direct Ferries** | `&children={N}&childAges={AGE1},{AGE2}` |
| **DFDS** | `&children={N}` |
| **Stena Line** | `&children={N}` |
| **P&O** | `&children={N}` |
| **Brittany Ferries** | `&children={N}&childAges={AGE1}` |

### Vehicle Parameters

When the user is travelling with a vehicle, append vehicle parameters where supported:
- **DFDS:** `&vehicleType=car`
- **Stena Line:** `&vehicle=car`
- **P&O:** `&vehicleType=car`
- **Direct Ferries:** `&vehicle=car`

**Link text example:** `[Book DFDS Dover→Calais 14:30 25 May](URL)` — always include operator, route, departure time, and date.

---

## ACCOMMODATION DEEP LINKS

| Platform | URL Template |
|----------|-------------|
| **Airbnb** | `https://www.airbnb.com/rooms/{LISTING_ID}?check_in={YYYY-MM-DD}&check_out={YYYY-MM-DD}&adults={N}&children={N}&infants={N}` |
| **Booking.com (property)** | `https://www.booking.com/hotel/{COUNTRY_CODE}/{HOTEL_SLUG}.html?checkin={YYYY-MM-DD}&checkout={YYYY-MM-DD}&group_adults={N}&group_children={N}&age={AGE1}&age={AGE2}&selected_currency={CURRENCY}` |
| **Booking.com (search)** | `https://www.booking.com/searchresults.html?ss={DESTINATION}&checkin={YYYY-MM-DD}&checkout={YYYY-MM-DD}&group_adults={N}&group_children={N}&age={AGE1}&age={AGE2}&selected_currency={CURRENCY}&nflt=review_score%3D80` |
| **Marriott** | `https://www.marriott.com/reservation/availabilitySearch.mi?propertyCode={CODE}&fromDate={MM/DD/YYYY}&toDate={MM/DD/YYYY}&flexibleDates=false&clusterCode=none&numberOfRooms=1&numberOfGuests={N}` |
| **Hilton** | `https://www.hilton.com/en/book/reservation/rooms/?ctyhocn={PROPERTY_CODE}&arrivalDate={YYYY-MM-DD}&departureDate={YYYY-MM-DD}&room1NumAdults={N}` |
| **IHG** | `https://www.ihg.com/hotels/us/en/find-hotels/hotel/rooms?qDest={HOTEL_NAME}&qCiD={DD}&qCiMy={MMYYYY}&qCoD={DD}&qCoMy={MMYYYY}&qAdlt={N}&qRms=1` |
| **Accor** | `https://all.accor.com/ssr/app/accor/rates/{HOTEL_ID}/index.en.shtml?dateIn={YYYY-MM-DD}&nights={N}&compositions={GUESTS}` |

### Children in Accommodation URLs

| Platform | How to add children |
|----------|-------------------|
| **Airbnb** | `&children={N}&infants={N}` (infants = under 2) |
| **Booking.com** | `&group_children={N}&age={AGE1}&age={AGE2}` (one `&age=` per child — required for pricing) |
| **Marriott** | `&childrenCount={N}` |
| **Hilton** | `&room1NumChildren={N}&room1Children1Age={AGE}` |

### Loyalty in Accommodation URLs

| Chain | How to include |
|-------|---------------|
| **Marriott Bonvoy** | Append `&memberNumber={NUMBER}` to Marriott URL |
| **Hilton Honors** | URL does not support pre-fill — say: "Log in to Hilton.com with your Honors account before booking to earn points" |
| **IHG One Rewards** | URL does not support pre-fill — say: "Log in to IHG with your One Rewards number before booking" |
| **Accor ALL** | URL does not support pre-fill — say: "Log in to all.accor.com with your ALL number before booking" |

**Link text example:** `[Book Villa Marina, Amalfi Coast](URL)` — always include property name and location. NEVER write "Search on Airbnb" or "Book this property".

---

## FALLBACK HIERARCHY

When constructing a booking link, follow this priority:

1. **Operator/airline direct deep link** (if template exists above) — best for loyalty, best price guarantee
2. **Aggregator deep link with route + dates** (Google Flights for air, Omio/Trainline EU for rail, Ferryhopper/Direct Ferries for sea, Booking.com for hotels) — still pre-filled, just not operator-direct
3. **Aggregator search link with route + dates** (broader search results) — acceptable fallback
4. **NEVER**: a bare homepage URL with no parameters

If you cannot construct a link with at least route and dates pre-filled, explain to the user: "I couldn't build a direct booking link for {operator} — here's the search link with your route pre-filled: {aggregator link}"

---

## LINK VERIFICATION

If the `agent-browser` tool is available, optionally verify that constructed URLs load the correct results before presenting them. If a link fails:
1. Try the aggregator fallback
2. If that also fails, provide the link with a note: "This link should pre-fill your search — if it doesn't load correctly, search manually for {route} on {date}"

---

## QUICK REFERENCE — Link Text Patterns

| Mode | Good link text | Bad link text |
|------|---------------|---------------|
| Flight | `Book Ryanair FR1234 STN→CTA 14 May` | `Search on Google Flights` |
| Flight | `Book BA LHR→FCO 14 May` | `Book this flight` |
| Train | `Book Eurostar 9014 London→Paris 08:01` | `Search on Eurostar` |
| Train | `Book on Trainline London→Edinburgh 09:30` | `Book this train` |
| Ferry | `Book DFDS Dover→Calais 14:30 25 May` | `Search on Ferryhopper` |
| Ferry | `Book Blue Star Piraeus→Santorini 07:25` | `Book this ferry` |
| Hotel | `Book Villa Marina, Amalfi Coast` | `Search on Airbnb` |
| Hotel | `Book Marriott Canary Wharf, London` | `Book this property` |
