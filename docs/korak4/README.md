# Korak 4 — Launch Preparation

Korak 4 je omejen na UI polish + operativno pripravljenost.  
Korak 3 ostane zamrznjen: brez backend core refaktorjev in brez schema sprememb.

## Release Checklist

1. `CI (main)` green.
2. `CI (nightly golden)` green.
3. `npm run smoke:enrich` uspe z real key-em (ključ se ne izpisuje).
4. `korak-3-production-hardened` tag obstaja.
5. `nearby_explore_v1`:
   - OFF by default potrjeno.
   - ON lokalno potrjeno.
6. `review_mock_mode`:
   - ON prikazuje Nearby Explore kartice brez GPS/network.
   - Ne vpliva na Surprise Me.
7. Secrets hygiene:
   - brez ključev v logih,
   - brez `.env` commitov.

## Feature Flags (dart-define)

Privzeto oba OFF.

```bash
flutter run \
  --dart-define=NEARBY_EXPLORE_V1=true \
  --dart-define=REVIEW_MOCK_MODE=true
```

## Reviewer Guidance (App Store / Play)

Za review build lahko vklopiš:
- `NEARBY_EXPLORE_V1=true`
- `REVIEW_MOCK_MODE=true`

S tem Nearby Explore vedno prikaže lokalni mock dataset (brez credentials/network), kar olajša review flow.

## Search A category mixing rule

Search bar A uporablja naslednje pravilo:
- `single`: Crystal Waters, Snow & Slopes, Into the Wild (ne smejo se kombinirati z drugimi)
- `mixable`: vse ostale kategorije (lahko se kombinirajo med seboj)

Konfliktno pravilo je `single wins`:
če je izbranih več mixable kategorij in uporabnik klikne single kategorijo, ostane izbrana samo ta single kategorija.
