<template>
  <ion-page class="page-root">
    <main class="dashboard">
      <header class="topbar">
        <a class="brand" href="#top" aria-label="CoinPulse home">
          <span class="brand-mark">C</span>
          <span>coin<span class="brand-accent">pulse</span></span>
        </a>
        <div class="live-status"><span class="live-dot"></span> Live market</div>
      </header>

      <section id="top" class="hero">
        <div class="eyebrow"><span class="sparkle">✦</span> MARKET OVERVIEW</div>
        <div class="hero-title-row">
          <div>
            <h1>Crypto market</h1>
            <p class="subtitle">Pantau aset digital teratas hari ini.</p>
          </div>
          <ion-button class="refresh-button" aria-label="Refresh data" @click="fetchCryptoData">
            <span class="refresh-icon" :class="{ spinning: loading }">↻</span>
            <span class="refresh-label">Refresh</span>
          </ion-button>
        </div>

        <div class="market-card">
          <div class="market-card-top">
            <div>
              <div class="metric-label">TOP 7 ASSETS</div>
              <div class="market-value">{{ cryptoList.length || '—' }}<span class="asset-suffix"> assets</span></div>
            </div>
            <div class="market-icon">◈</div>
          </div>
          <div class="market-card-bottom">
            <span class="market-caption">Data harga dalam US Dollar</span>
            <span class="updated"><span class="live-dot"></span> {{ loading ? 'Memuat data…' : 'Terkini' }}</span>
          </div>
          <div class="card-glow"></div>
        </div>
      </section>

      <section class="assets-section">
        <div class="section-heading">
          <div><h2>Aset teratas</h2><p>Market cap terbesar</p></div>
          <span class="asset-count">{{ cryptoList.length }} COINS</span>
        </div>

        <div v-if="loading && !cryptoList.length" class="state-card">
          <ion-spinner name="crescent" color="primary"></ion-spinner>
          <p>Mengambil data pasar…</p>
        </div>
        <div v-else-if="error" class="state-card error-state">
          <span class="state-icon">!</span>
          <p>{{ errorMessage }}</p>
          <ion-button size="small" fill="outline" @click="fetchCryptoData">Coba lagi</ion-button>
        </div>
        <div v-else class="asset-list">
          <article v-for="(coin, index) in cryptoList" :key="coin.id" class="asset-row">
            <div class="coin-avatar" :class="`coin-${index % 6}`">{{ coin.symbol.slice(0, 1) }}</div>
            <div class="coin-info"><div class="coin-name">{{ coin.name }}</div><div class="coin-meta"><span class="coin-symbol">{{ coin.symbol }}</span><span class="rank-pill">#{{ coin.rank }}</span></div></div>
            <div class="coin-price"><div class="price-value">{{ formatPrice(coin.price_usd) }}</div><div class="price-currency">USD</div></div>
            <div class="trend-mark" aria-label="Market asset">↗</div>
          </article>
        </div>
        <div class="source-note"><span class="source-dot"></span> Data disediakan oleh Coinlore API</div>
      </section>
      <footer class="footer"><span>COINPULSE</span><span>Market data at a glance</span></footer>
    </main>
  </ion-page>
</template>

<script setup lang="ts">
import { ref, onMounted } from 'vue';
import { IonPage, IonButton, IonSpinner } from '@ionic/vue';

interface CryptoCoin {
  id: number;
  rank: number;
  name: string;
  symbol: string;
  price_usd: string;
}

const cryptoList = ref<CryptoCoin[]>([]);
const loading = ref(true);
const error = ref(false);
const errorMessage = ref('');

const fetchCryptoData = async () => {
  loading.value = true;
  error.value = false;
  errorMessage.value = '';
  try {
    const response = await fetch('https://api.coinlore.com/api/tickers/');
    if (!response.ok) throw new Error('Gagal mengambil data dari Coinlore');
    const result = await response.json();
    const data = Array.isArray(result.data) ? result.data : [];
    cryptoList.value = data.slice(0, 7).map((coin: any) => ({
      id: Number.isFinite(Number(coin.id)) ? Number(coin.id) : Number(coin.rank),
      rank: Number(coin.rank),
      name: String(coin.name),
      symbol: String(coin.symbol),
      price_usd: String(coin.price_usd),
    }));
  } catch (err: any) {
    error.value = true;
    errorMessage.value = err?.message || 'Terjadi kesalahan saat memuat data';
  } finally {
    loading.value = false;
  }
};

const formatPrice = (value: string) => {
  const num = Number(value);
  return Number.isFinite(num)
    ? new Intl.NumberFormat('en-US', { minimumFractionDigits: 2, maximumFractionDigits: 2 }).format(num)
    : value;
};

onMounted(fetchCryptoData);
</script>

<style>
:root { color-scheme: light; font-family: Inter, ui-sans-serif, system-ui, -apple-system, BlinkMacSystemFont, "Segoe UI", sans-serif; background: #f5f7fb; }
* { box-sizing: border-box; }
body { margin: 0; background: #f5f7fb; color: #182235; }
.page-root { --ion-background-color: #f5f7fb; min-height: 100vh; }
.dashboard { width: min(100%, 900px); min-height: 100vh; margin: 0 auto; padding: 25px 32px 24px; }
.topbar { height: 44px; display: flex; align-items: center; justify-content: space-between; }
.brand { display: inline-flex; align-items: center; gap: 10px; color: #182235; text-decoration: none; font-size: 17px; font-weight: 800; letter-spacing: -.7px; }
.brand-mark { width: 30px; height: 30px; display: grid; place-items: center; border-radius: 10px; color: white; background: linear-gradient(145deg,#7058ed,#4835c4); font-size: 16px; box-shadow: 0 5px 12px #5742ce38; }
.brand-accent { color: #6854df; }
.live-status { display: flex; align-items: center; gap: 7px; color: #707c90; font-size: 11px; font-weight: 650; }
.live-dot,.source-dot { width: 7px; height: 7px; display: inline-block; border-radius: 50%; background: #34c98b; box-shadow: 0 0 0 3px #34c98b20; }
.hero { margin-top: 40px; }
.eyebrow { color: #7867da; font-size: 10px; font-weight: 800; letter-spacing: 1.5px; }
.sparkle { margin-right: 5px; font-size: 13px; }
.hero-title-row { display: flex; align-items: center; justify-content: space-between; gap: 16px; margin: 10px 0 23px; }
h1 { margin: 0; color: #172136; font-size: clamp(27px, 5vw, 35px); line-height: 1.15; letter-spacing: -1.4px; }
.subtitle { margin: 8px 0 0; color: #8892a3; font-size: 13px; }
.refresh-button { --background: #fff; --background-hover: #f5f3ff; --color: #5946cc; --border-radius: 11px; --box-shadow: 0 5px 16px #222f4810; height: 39px; margin: 0; font-size: 12px; font-weight: 700; text-transform: none; }
.refresh-icon { margin-right: 6px; font-size: 18px; line-height: 1; }
.spinning { display: inline-block; animation: spin 1s linear infinite; }
@keyframes spin { to { transform: rotate(360deg); } }
.market-card { position: relative; isolation: isolate; overflow: hidden; min-height: 130px; padding: 20px 23px 16px; border-radius: 17px; color: #fff; background: linear-gradient(112deg,#6450dc 0%,#5645c8 58%,#4435b2 100%); box-shadow: 0 13px 24px #4e3dc426; }
.market-card-top,.market-card-bottom { position: relative; z-index: 1; display: flex; justify-content: space-between; align-items: center; }
.metric-label { color: #d5d0ff; font-size: 9px; letter-spacing: 1.3px; font-weight: 750; }
.market-value { margin-top: 6px; font-size: 29px; line-height: 1; letter-spacing: -1px; font-weight: 750; }
.asset-suffix { color: #ddd9ff; font-size: 13px; font-weight: 500; letter-spacing: 0; }
.market-icon { display: grid; place-items: center; width: 42px; height: 42px; border: 1px solid #ffffff3a; border-radius: 13px; color: #e6e2ff; background: #ffffff17; font-size: 25px; }
.market-card-bottom { margin-top: 16px; padding-top: 12px; border-top: 1px solid #ffffff28; }
.market-caption { color: #dedaff; font-size: 10px; }
.updated { display: flex; align-items: center; gap: 7px; color: #f3f1ff; font-size: 10px; }
.updated .live-dot { width: 6px; height: 6px; background: #77ebbd; box-shadow: 0 0 0 3px #77ebbd22; }
.card-glow { position: absolute; z-index: 0; width: 180px; height: 180px; top: -100px; right: 90px; border: 1px solid #ffffff18; border-radius: 50%; box-shadow: 0 0 0 22px #ffffff08,0 0 0 45px #ffffff05; }
.assets-section { margin-top: 30px; }
.section-heading { display: flex; align-items: center; justify-content: space-between; margin-bottom: 13px; }
h2 { margin: 0; font-size: 17px; letter-spacing: -.45px; }
.section-heading p { margin: 4px 0 0; color: #8d97a7; font-size: 11px; }
.asset-count { color: #919aab; font-size: 9px; letter-spacing: 1px; font-weight: 750; }
.asset-list { overflow: hidden; border: 1px solid #e9edf4; border-radius: 15px; background: #fff; box-shadow: 0 7px 22px #202e4708; }
.asset-row { display: grid; grid-template-columns: 38px minmax(0,1fr) minmax(98px,auto) 20px; align-items: center; gap: 12px; min-height: 62px; padding: 9px 15px; border-bottom: 1px solid #eff1f5; }
.asset-row:last-child { border-bottom: 0; }
.coin-avatar { width: 35px; height: 35px; display: grid; place-items: center; border-radius: 12px; color: #fff; font-size: 15px; font-weight: 750; background: #f4a72d; }
.coin-1 { color: #20344c; background: #eaf0f6; }.coin-2 { background: #7186ed; }.coin-3 { color: #3c2e12; background: #f3d77b; }.coin-4 { background: #71809a; }.coin-5 { background: #37a981; }
.coin-name { overflow: hidden; color: #202b3d; font-size: 12px; font-weight: 700; text-overflow: ellipsis; white-space: nowrap; }
.coin-meta { display: flex; align-items: center; gap: 7px; margin-top: 4px; }
.coin-symbol { color: #8d97a7; font-size: 9px; font-weight: 700; letter-spacing: .5px; }
.rank-pill { padding: 2px 5px; border-radius: 5px; color: #7b6bd3; background: #f1efff; font-size: 8px; font-weight: 700; }
.coin-price { text-align: right; }
.price-value { color: #202b3d; font-size: 12px; font-weight: 720; font-variant-numeric: tabular-nums; }
.price-currency { margin-top: 3px; color: #9ba4b2; font-size: 8px; letter-spacing: .8px; }
.trend-mark { color: #33b880; font-size: 17px; text-align: right; }
.state-card { min-height: 180px; display: flex; flex-direction: column; align-items: center; justify-content: center; gap: 9px; padding: 22px; border: 1px solid #e9edf4; border-radius: 15px; color: #8792a4; background: #fff; text-align: center; font-size: 12px; }
.state-card p { margin: 2px 0; }.error-state { color: #bf4855; }.state-icon { display: grid; place-items: center; width: 28px; height: 28px; border-radius: 50%; background: #fff0f1; font-weight: 800; }
.source-note { display: flex; align-items: center; gap: 8px; margin: 13px 2px 0; color: #9aa3b2; font-size: 9px; }
.source-dot { width: 5px; height: 5px; box-shadow: none; }
.footer { display: flex; justify-content: space-between; margin-top: 31px; padding-top: 14px; border-top: 1px solid #e8ebf1; color: #a3abba; font-size: 9px; }.footer span:first-child { font-weight: 800; letter-spacing: 1.2px; }
.page-root { display: grid; place-items: center; min-height: 100vh; padding: 26px 16px; background: radial-gradient(ellipse at 50% 0%, #e9e7fb 0%, #f5f7fb 52%); }
.dashboard { position: relative; width: min(100%, 410px); height: min(860px, calc(100vh - 52px)); min-height: 560px; padding: 30px 22px 20px; overflow-x: hidden; overflow-y: auto; scrollbar-width: thin; scrollbar-color: #d2cdef transparent; border: 9px solid #202535; border-radius: 43px; background: #f5f7fb; box-shadow: 0 28px 70px #1d254130, 0 4px 12px #1d25411c; }
.dashboard::before { content: ""; position: absolute; z-index: 3; top: 8px; left: 50%; width: 76px; height: 17px; transform: translateX(-50%); border-radius: 0 0 12px 12px; background: #202535; }
.dashboard::after { content: ""; position: absolute; z-index: 2; top: 13px; left: calc(50% + 18px); width: 5px; height: 5px; border-radius: 50%; background: #353d52; }
@media (max-width: 540px) {
  .page-root { display: block; padding: 0; background: #f5f7fb; }
  .dashboard { width: 100%; height: auto; min-height: 100vh; min-height: 100dvh; overflow: visible; padding: 20px 18px 22px; border: 0; border-radius: 0; box-shadow: none; }
  .dashboard::before, .dashboard::after { display: none; }
 .dashboard { padding: 18px 18px 22px; }.hero { margin-top: 31px; }.hero-title-row { margin-bottom: 19px; }.refresh-button { height: 36px; --padding-start: 11px; --padding-end: 11px; }.asset-row { grid-template-columns: 35px minmax(0,1fr) minmax(82px,auto) 14px; gap: 9px; min-height: 60px; padding: 8px 11px; }.price-value { font-size: 11px; } }
</style>
