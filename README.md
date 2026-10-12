# expo-crypto-switch

A crypto swap switcher with live CoinGecko prices, bidirectional amount input, and fluid motion throughout.

<!-- HERO GIF PLACEHOLDER — add hero gif here after push -->
<!-- e.g. <img width="1280" height="720" alt="expo-crypto-switch demo" src="PASTE_URL_HERE" /> -->

---

## ✨ Features

- 💱 **Bidirectional amount input** — edit either the pay or receive field; the opposite amount updates live from the current quote
- 🔄 **Spring-physics card swap** — pay/receive cards animate past each other with an in-flight guard plus a disabled swap button so rapid taps never corrupt state
- 🌍 **Live crypto prices** — fetches USD prices from the [CoinGecko simple price API](https://docs.coingecko.com/v3.0.1/reference/simple-price) out of the box, with a 15-minute in-memory cache and stale-request cancellation
- 🔌 **Injectable price source** — skip the network entirely by passing `prices` directly as a prop; or swap out CoinGecko for any backend by passing a `fetcher` to `useCryptoPrice`
- 🎭 **Morph-transition picker** — the coin list springs open from the measured card bounds and collapses back on dismiss, carrying a greyed quote veil so the halves never go blank
- 🪙 **Coin icons with fallback** — icons load from the CoinCap CDN, are prefetched on mount, and fall back to a letter tile if a load fails
- 🌗 **Animated light/dark theme** — every surface and text crossfades on the UI thread against one shared progress value; works uncontrolled out of the box or controlled via `theme` + `onThemeChange`
- 💵 **USD valuations** — each card shows its USD value under the amount, with the `spread` applied to the receive side
- 📏 **Crypto-grade precision** — up to 8 decimal places, trailing zeros trimmed, comma-as-decimal handled for locale keyboards, input capped at 21 chars
- 🧠 **TypeScript-first** — fully typed props, `CryptoPriceFetcher` as a named injectable type, structured `onSend` detail object

---

## ⚙️ Installation

This isn't published as an npm package yet — copy the source directly into your project.

```bash
git clone https://github.com/ManasCodeXart/expo-crypto-switch.git
```

Copy `components/`, `constants/`, `hooks/`, `providers/`, and `utils/` from `src/` into your project, then install the peer dependencies:

```bash
npx expo install react-native-reanimated react-native-worklets react-native-svg expo-image
```

> Reanimated 4.x ships its worklets runtime as the separate `react-native-worklets` package — it's required alongside `react-native-reanimated`, not optional.

> Requires `react-native-reanimated`'s Babel plugin already configured. `expo-image` is required for the coin icons.

---

## 🚀 Usage

```tsx
import { CryptoSwitch } from './components/CryptoSwitch';

export function SwapScreen() {
  return (
    <CryptoSwitch
      onCoinChange={(from, to) => console.log(from.symbol, '→', to.symbol)}
      onSend={(details) => console.log('Send', details)}
    />
  );
}
```

### Bring your own price source

```tsx
// Pass static or externally-fetched USD prices — skips the CoinGecko fetch
// for every symbol present in the map
<CryptoSwitch
  prices={{ BTC: 97000, ETH: 3400 }}
  onSend={(details) => sendSwap(details)}
/>
```

```tsx
// Or inject a custom fetcher (e.g. your own backend) directly into the hook
import { useCryptoPrice } from './hooks/useCryptoPrice';

const { priceUsd, isLoading, error, refresh } = useCryptoPrice('BTC', {
  fetcher: (symbol) => myApi.getUsdPrice(symbol),
  cacheTtlMs: 5 * 60 * 1000,
});
```

## Preview

<!-- PREVIEW VIDEO PLACEHOLDER — add preview video here after push -->

---

## 🧱 Component Anatomy

```
<CryptoSwitch>
  ├─ CoinPicker     (morph-transition coin list)
  │   └─ CoinRow    (icon + symbol + name + selection row)
  ├─ CoinPill       (icon + symbol pill that opens the picker)
  ├─ ThemedSwapIcon (bidirectional arrow icon)
  └─ ThemeToggleIcon (light/dark toggle icon)
```

`CoinPicker`, `CoinPill`, and `CoinRow` are also exported individually if you need them outside the switcher.

---

## 🧩 API

### `<CryptoSwitch>`

| Prop | Type | Default | Description |
|---|---|---|---|
| `coins` | `readonly Coin[]` | built-in list (BTC, ETH, HYPE, ZEC, QNT, SOL, DOGE) | The coin list shown in both pickers. |
| `defaultFromCoin` | `Coin` | `BTC` | Coin shown in the pay card on first render. Falls back to `coins[0]`. |
| `defaultToCoin` | `Coin` | `ETH` | Coin shown in the receive card on first render. Falls back to `coins[0]`. |
| `defaultSendAmount` | `string` | `"1"` | Amount pre-filled in the pay field on first render. |
| `spread` | `number` | `0.003` | Fraction taken off the quote (0.003 = 0.3%). Applied to the effective rate and the receive-side USD value. Same-coin pairs quote at 1 with no spread on the rate. |
| `prices` | `Readonly<Record<string, number>>` | — | Pass explicit USD prices (keyed by symbol, any casing) to skip the internal CoinGecko fetch for those symbols. |
| `swapIcon` | `ReactNode` | built-in `<ThemedSwapIcon />` | Replace the swap button icon with any node. Also forwarded into the picker. |
| `style` | `StyleProp<ViewStyle>` | — | Additional styles applied to the root container. |
| `theme` | `"light" \| "dark"` | — | Controlled theme. Omit for uncontrolled (starts light, toggles internally). |
| `onAmountChange` | `(amount: string) => void` | — | Fires on every keystroke with the implied send-side amount as a string. |
| `onCoinChange` | `(from: Coin, to: Coin) => void` | — | Fires when either coin is changed or the cards are swapped. |
| `onSend` | `(details: { amount: number; from: Coin; to: Coin }) => void` | — | Fires on Send tap with the current send amount and both coins. |
| `onThemeChange` | `(theme: ThemeName) => void` | — | Fires on theme toggle with the next theme. |

### `<CoinPicker>`

| Prop | Type | Default | Description |
|---|---|---|---|
| `target` | `"from" \| "to"` | — | Which card opened the picker. Drives the header title when used inside the switcher. |
| `title` | `string` | — | Header title shown above the list. |
| `bounds` | `CardBounds` | — | Measured heights of the two cards, used to anchor the morph animation. |
| `selectedSymbol` | `string` | — | The currently selected coin symbol. Highlighted in the list. |
| `coins` | `readonly Coin[]` | — | The full list to render. |
| `swapIcon` | `ReactNode` | built-in icon | Icon rendered in the center during the morph. |
| `veil` | `QuoteVeil` | — | Preformatted quote strings shown greyed on the morph halves while they spring back. |
| `themeProgress` | `SharedValue<number>` | — | Shared 0 (light) → 1 (dark) progress driving the themed styles. |
| `onSelect` | `(coin: Coin) => void` | — | Fired once per selection; double-select is guarded internally. |
| `onClose` | `() => void` | — | Fired after the exit morph settles. |

### `useCryptoPrice`

```ts
const { priceUsd, isLoading, error, refresh } = useCryptoPrice(symbol, options);
```

| Option | Type | Default | Description |
|---|---|---|---|
| `fetcher` | `CryptoPriceFetcher` | CoinGecko | Any `(symbol) => Promise<number>` function returning the USD price. |
| `cacheTtlMs` | `number` | `900000` (15 min) | How long a cached price is considered fresh before the next mount triggers a refetch. |
| `enabled` | `boolean` | `true` | Set to `false` to skip fetching entirely — used internally when a `prices` override is provided. |

### Types

```ts
interface Coin {
  readonly symbol: string;
  readonly name: string;
  readonly coingeckoId: string; // CoinGecko id used for price lookup
}

type CryptoPriceFetcher = (symbol: string) => Promise<number>;

interface CardBounds {
  readonly topHeight: number;
  readonly bottomHeight: number;
  readonly unionHeight: number;
}

interface QuoteVeil {
  readonly sendAmount: string;
  readonly receiveAmount: string;
  readonly payUsd: string;
  readonly receiveUsd: string;
}

type ThemeName = "light" | "dark";
```

---

## 🔤 Fonts

Text elements use the **Space Grotesk** family (`SpaceGroteskMedium`, `SpaceGroteskSemiBold`, `SpaceGroteskBold`) by name. If you don't load these yourself via `expo-font` / `useFonts`, React Native falls back to the system font silently — everything still works, you'll just get system-font weights instead of Space Grotesk. Load the family under those exact names if you want the intended look.

---

## 📄 License

MIT — see [LICENSE](./LICENSE).

---

## 🧱 Stack

[Expo SDK 57](https://expo.dev/changelog) · [React Native 0.86](https://reactnative.dev/) · [Reanimated 4.5](https://docs.swmansion.com/react-native-reanimated/) · [React Native Worklets 0.10](https://docs.swmansion.com/react-native-reanimated/) · [react-native-svg 15.15](https://github.com/software-mansion/react-native-svg) · [expo-image](https://docs.expo.dev/versions/latest/sdk/image/)
