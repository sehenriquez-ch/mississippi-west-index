#!/usr/bin/env python3
"""
Mississippi Divide West Index v11.3 — Level Fix + Kurtosis Diagnostic
=============================================================================
CAMBIOS v11.3 sobre v11.2:

1. FIX CRITICO: calculate_index() ahora calcula el Level del indice como el
   cumulative product de los retornos equal-weight (base 1000), en vez de usar
   el divisor basado en float-adjusted market-cap. Antes el Level y los retornos
   reportados eran DOS INDICES DISTINTOS. Ahora son matematicamente consistentes.

2. NUEVO: funcion diagnostico_kurtosis() para identificar la fuente de outliers.

Todo lo demas (Newey-West, batches, metricas, graficos, reporte) se mantiene igual.
"""

import argparse
import logging
import sqlite3
import hashlib
import pickle
import io
import time
import sys
import warnings
from datetime import datetime, timedelta
from pathlib import Path
from enum import Enum
from typing import Dict, List, Optional, Any, Set
from collections import OrderedDict

import pandas as pd
import numpy as np
import matplotlib.pyplot as plt
import matplotlib.ticker as mticker
import yfinance as yf
import requests
from requests.adapters import HTTPAdapter
from scipy import stats as sp_stats
try:
    import statsmodels.api as sm
    HAS_STATSMODELS = True
except ImportError:
    HAS_STATSMODELS = False
    print("⚠️  statsmodels no instalado. Corre: !pip install statsmodels -q")
    print("   Sin statsmodels, el ajuste Newey-West no se puede calcular y se")
    print("   usara OLS simple como fallback (menos riguroso).")

warnings.filterwarnings('ignore')

# ================================================================
# CONSTANTES DEL INDICE
# ================================================================

class Region(str, Enum):
    EAST    = "Este"
    WEST    = "Oeste"
    NEUTRAL = "Neutral"

class Side(str, Enum):
    WEST = "$WEST"

TRADING_DAYS_PER_YEAR = 252
MIN_ADV_USD           = 500_000
CAP_POSITION          = 0.15
CAP_SECTOR            = 0.25
MAX_ITERATIONS        = 10
MAX_COMPONENTS        = 100
REBALANCE_MONTHS      = [3, 6, 9, 12]
REBALANCE_WEEKDAY     = 4
REBALANCE_ORDINAL     = 3
SELECTION_DAYS_BEFORE = 5
ANNOUNCE_DAYS_BEFORE  = 5

SP500_HIST_URL = (
    "https://raw.githubusercontent.com/fja05680/sp500/master/"
    "S%26P%20500%20Historical%20Components%20%26%20Changes%20(Updated).csv"
)
WIKIPEDIA_URL = "https://en.wikipedia.org/wiki/List_of_S%26P_500_companies"

MAPA_ESTADOS = {
    'NY':'Este','NJ':'Este','PA':'Este','MA':'Este','CT':'Este','MD':'Este',
    'VA':'Este','NC':'Este','SC':'Este','GA':'Este','FL':'Este','OH':'Este',
    'IL':'Este','MI':'Este','IN':'Este','WI':'Este','KY':'Este','TN':'Este',
    'WV':'Este','DE':'Este','VT':'Este','NH':'Este','ME':'Este','RI':'Este',
    'AL':'Este','MS':'Este','DC':'Este',
    'CA':'Oeste','WA':'Oeste','OR':'Oeste','TX':'Oeste','AZ':'Oeste','CO':'Oeste',
    'UT':'Oeste','NV':'Oeste','ID':'Oeste','MT':'Oeste','WY':'Oeste','NM':'Oeste',
    'AK':'Oeste','HI':'Oeste','MN':'Oeste','IA':'Oeste','MO':'Oeste','AR':'Oeste',
    'LA':'Oeste','ND':'Oeste','SD':'Oeste','NE':'Oeste','KS':'Oeste','OK':'Oeste',
    'New York':'Este','New Jersey':'Este','Pennsylvania':'Este',
    'Massachusetts':'Este','Connecticut':'Este','Maryland':'Este',
    'Virginia':'Este','North Carolina':'Este','South Carolina':'Este',
    'Georgia':'Este','Florida':'Este','Ohio':'Este','Illinois':'Este',
    'Michigan':'Este','Indiana':'Este','Wisconsin':'Este','Kentucky':'Este',
    'Tennessee':'Este','West Virginia':'Este','Delaware':'Este',
    'Vermont':'Este','New Hampshire':'Este','Maine':'Este','Rhode Island':'Este',
    'Alabama':'Este','Mississippi':'Este','District of Columbia':'Este',
    'California':'Oeste','Washington':'Oeste','Oregon':'Oeste','Texas':'Oeste',
    'Arizona':'Oeste','Colorado':'Oeste','Utah':'Oeste','Nevada':'Oeste',
    'Idaho':'Oeste','Montana':'Oeste','Wyoming':'Oeste','New Mexico':'Oeste',
    'Alaska':'Oeste','Hawaii':'Oeste','Minnesota':'Oeste','Iowa':'Oeste',
    'Missouri':'Oeste','Arkansas':'Oeste','Louisiana':'Oeste',
    'North Dakota':'Oeste','South Dakota':'Oeste','Nebraska':'Oeste',
    'Kansas':'Oeste','Oklahoma':'Oeste',
}

HQ_CORRECTIONS = {
    'TSLA': [(None, '2021-12-01', 'CA'), ('2021-12-01', None, 'TX')],
    'ORCL': [(None, '2020-12-01', 'CA'), ('2020-12-01', None, 'TX')],
    'HPQ':  [(None, '2020-12-01', 'CA'), ('2020-12-01', None, 'TX')],
    'SCHW': [(None, '2021-01-01', 'CA'), ('2021-01-01', None, 'TX')],
    'CBRE': [(None, '2020-01-01', 'CA'), ('2020-01-01', None, 'TX')],
    'TMUS': [(None, None, 'WA')],
}

SEDES_HISTORICAS_OESTE = {
    'ATVI':'CA','BHGE':'TX','COG':'TX','FLIR':'OR','HCP':'CA','IGT':'NV',
    'ILMN':'CA','LLTC':'CA','LVLT':'CO','MAC':'CA','MAT':'CA','MRO':'TX',
    'NBL':'TX','NBR':'TX','NEM':'CO','NFX':'TX','NOV':'TX','NTAP':'CA',
    'O':'CA','OXY':'TX','PCL':'WA','PLD':'CA','PXD':'TX','QCOM':'CA',
    'RHI':'CA','ROST':'CA','RRC':'TX','RSG':'AZ','SBUX':'WA','SCHW':'TX',
    'SNDK':'CA','SRE':'CA','SWN':'TX','SYMC':'CA','SYY':'TX','TDC':'CA',
    'THC':'TX','TMK':'TX','TSO':'TX','TXN':'TX','UDR':'CO','V':'CA',
    'VAR':'CA','VFC':'CO','VLO':'TX','WDC':'CA','WFC':'CA','WFM':'TX',
    'WM':'TX','WMB':'OK','WU':'CO','WY':'WA','WYNN':'NV','XEC':'OK',
    'XLNX':'CA','XOM':'TX','YHOO':'CA','ZION':'UT',
}
SEDES_HISTORICAS_ESTE = {
    'CELG':'NJ','ARNC':'PA','CTXS':'FL','DISCA':'MD','DISCK':'MD','DPS':'TX',
    'DWDP':'MI','EMC':'MA','ESRX':'MO','ETFC':'NJ','GGP':'IL','HAR':'NY',
    'HES':'NY','HOG':'WI','HOT':'MD','HRS':'FL','HTZ':'FL','JOY':'WI',
    'KORS':'NY','KRFT':'IL','LB':'OH','LEG':'MO','LLL':'VA','LM':'NY',
    'LO':'NC','M':'OH','MDLZ':'IL','MHK':'GA','MJN':'IL','MKC':'MD',
    'MLM':'NC','MMC':'NY','MON':'MO','MOS':'FL','MUR':'AR','MYL':'PA',
    'NLSN':'NY','NOC':'VA','NUE':'NC','NWL':'NJ','PAYX':'NY','PBCT':'CT',
    'PCLN':'CT','PKI':'MA','PM':'NY','PNC':'PA','PPG':'PA','PPL':'PA',
    'PRU':'NJ','PSA':'CA','PSX':'TX','PVH':'NY','PWR':'TX','PX':'CT',
    'R':'FL','RAI':'NC','RCL':'FL','REGN':'NY','RF':'AL','RHT':'NC',
    'RL':'NY','ROK':'WI','ROP':'FL','ROST':'CA','RRC':'TX','RSG':'AZ',
    'RTN':'MA','SCG':'SC','SCHW':'TX','SEE':'NC','SHW':'OH','SIAL':'MO',
    'SIG':'OH','SJM':'OH','SLB':'TX','SNA':'WI','SNDK':'CA','SNI':'TN',
    'SO':'GA','SPG':'IN','SPLS':'MA','SRCL':'IL','STI':'GA','STJ':'MN',
    'STT':'MA','STZ':'NY','SWK':'CT','SWKS':'MA','SWN':'TX','SYF':'CT',
    'SYK':'MI','TAP':'CO','TDG':'FL','TE':'MA','TEG':'MI','TGNA':'VA',
    'TGT':'MN','TIF':'NY','TJX':'MA','TMO':'MA','TRIP':'MA','TROW':'MD',
    'TRV':'CT','TSCO':'TN','TSN':'AR','TSS':'GA','TWX':'NY','TXT':'RI',
    'UA':'MD','UAA':'MD','UAL':'IL','UDR':'CO','UHS':'PA','UNH':'MN',
    'UNM':'ME','UNP':'NE','UPS':'GA','URBN':'PA','URI':'CT','USB':'MN',
    'UTX':'CT','VIAB':'NY','VMC':'VA','VNO':'NY','VRSK':'NJ','VRSN':'VA',
    'VRTX':'MA','VTR':'IL','VZ':'NY','WAT':'MA','WBA':'IL','WEC':'WI',
    'WHR':'MI','WMT':'AR','WRK':'GA','WYN':'NJ','XEL':'MN','XL':'NY',
    'XRAY':'PA','XRX':'CT','XYL':'NY','YUM':'KY','ZBH':'IN','ZTS':'NJ',
}

TICKER_TO_YAHOO = {
    'BRK.B': 'BRK-B', 'BF.B': 'BF-B',
}

class TickerNormalizer:
    @staticmethod
    def to_yahoo(ticker: str) -> str:
        ticker = ticker.strip().upper()
        if ticker.startswith("$"):
            ticker = ticker[1:]
        if ticker in TICKER_TO_YAHOO:
            return TICKER_TO_YAHOO[ticker]
        if "." in ticker:
            return ticker.replace(".", "-")
        return ticker

class MississippiDivideError(Exception): pass
class DataError(MississippiDivideError): pass
class DataProviderError(DataError): pass
class LookAheadBiasError(DataError): pass

class CacheManager:
    def __init__(self, db_path: Path, default_ttl_hours: int = 24):
        self.db_path = db_path
        self.default_ttl = timedelta(hours=default_ttl_hours)
        self._init_db()

    def _init_db(self):
        self.db_path.parent.mkdir(parents=True, exist_ok=True)
        with sqlite3.connect(self.db_path) as conn:
            conn.execute("""
                CREATE TABLE IF NOT EXISTS cache (
                    key TEXT PRIMARY KEY, value BLOB,
                    created_at TIMESTAMP, expires_at TIMESTAMP, data_type TEXT
                )
            """)
            conn.execute("CREATE INDEX IF NOT EXISTS idx_expires ON cache(expires_at)")

    def _hash(self, key: str) -> str:
        return hashlib.sha256(key.encode()).hexdigest()

    def get(self, key: str) -> Optional[Any]:
        try:
            h = self._hash(key)
            with sqlite3.connect(self.db_path) as conn:
                row = conn.execute(
                    "SELECT value, data_type, expires_at FROM cache WHERE key=?", (h,)
                ).fetchone()
                if row is None:
                    return None
                value, dtype, expires = row
                if datetime.now() > datetime.fromisoformat(expires):
                    conn.execute("DELETE FROM cache WHERE key=?", (h,))
                    return None
                if dtype == "df":
                    return pd.read_parquet(io.BytesIO(value))
                return pickle.loads(value)
        except Exception:
            return None

    def set(self, key: str, value: Any, ttl_hours: Optional[int] = None):
        try:
            h = self._hash(key)
            ttl = timedelta(hours=ttl_hours) if ttl_hours else self.default_ttl
            expires = datetime.now() + ttl
            if isinstance(value, pd.DataFrame):
                buf = io.BytesIO()
                value.to_parquet(buf, index=True)
                blob, dtype = buf.getvalue(), "df"
            else:
                blob, dtype = pickle.dumps(value), "pickle"
            with sqlite3.connect(self.db_path) as conn:
                conn.execute(
                    "INSERT OR REPLACE INTO cache VALUES (?,?,?,?,?)",
                    (h, blob, datetime.now(), expires, dtype)
                )
        except Exception as e:
            logging.warning(f"Cache write error: {e}")

class YahooProvider:
    MAX_RETRIES = 3
    BASE_DELAY  = 2.0
    BATCH_SIZE  = 20

    def __init__(self, cache_manager=None):
        self.cache = cache_manager
        self.session = requests.Session()
        adapter = HTTPAdapter(max_retries=3, pool_connections=10, pool_maxsize=20)
        self.session.mount("https://", adapter)
        self._ticker_cache: Dict[str, yf.Ticker] = {}
        self._failed_tickers: Set[str] = set()
        self._delisted_tickers: Set[str] = set()

    def _get_ticker(self, symbol: str) -> yf.Ticker:
        if symbol not in self._ticker_cache:
            self._ticker_cache[symbol] = yf.Ticker(symbol)
        return self._ticker_cache[symbol]

    def _retry(self, func, *args, **kwargs):
        for attempt in range(self.MAX_RETRIES):
            try:
                return func(*args, **kwargs)
            except Exception as e:
                if attempt == self.MAX_RETRIES - 1:
                    raise DataProviderError(f"Fallo tras {self.MAX_RETRIES} intentos: {e}")
                delay = self.BASE_DELAY * (2 ** attempt) + np.random.uniform(0, 1)
                logging.warning(f"Reintentando en {delay:.1f}s...")
                time.sleep(delay)

    def get_historical_prices(self, tickers, start, end):
        cache_key = f"prices_v2_{hash(tuple(sorted(tickers)))}_{start.date()}_{end.date()}"
        if self.cache:
            cached = self.cache.get(cache_key)
            if cached is not None:
                logging.info("Usando precios desde cache")
                return cached

        logging.info(f"Descargando {len(tickers)} tickers en batches de {self.BATCH_SIZE}...")
        logging.info("NOTA: Tickers delistados generaran warnings. Esto es NORMAL.")

        all_close, all_volume = [], []
        tickers_list = sorted(list(tickers))
        total_batches = (len(tickers_list) + self.BATCH_SIZE - 1) // self.BATCH_SIZE

        for i in range(0, len(tickers_list), self.BATCH_SIZE):
            batch = tickers_list[i:i + self.BATCH_SIZE]
            batch_num = i // self.BATCH_SIZE + 1
            logging.info(f"  Batch {batch_num}/{total_batches}: {batch[0]}...{batch[-1]}")

            try:
                data = self._retry(
                    yf.download,
                    tickers=batch, start=start, end=end,
                    auto_adjust=True, progress=False, threads=True, timeout=30
                )
                if data is None or data.empty:
                    continue

                if isinstance(data.columns, pd.MultiIndex):
                    df_close = data['Close'].copy() if 'Close' in data.columns.get_level_values(0) else pd.DataFrame()
                    df_vol = data['Volume'].copy() if 'Volume' in data.columns.get_level_values(0) else pd.DataFrame()
                else:
                    df_close = data[['Close']].copy() if 'Close' in data.columns else pd.DataFrame()
                    if not df_close.empty:
                        df_close.columns = [batch[0]]
                    df_vol = data[['Volume']].copy() if 'Volume' in data.columns else pd.DataFrame()
                    if not df_vol.empty:
                        df_vol.columns = [batch[0]]

                if not df_close.empty:
                    all_close.append(df_close.ffill().dropna(axis=1, how='all'))
                if not df_vol.empty:
                    all_volume.append(df_vol.ffill().bfill())

            except Exception as e:
                logging.warning(f"  Batch {batch_num} fallo completo ({e}), reintentando ticker por ticker...")
                for t in batch:
                    try:
                        single = yf.download(t, start=start, end=end,
                                              auto_adjust=True, progress=False, timeout=15)
                        if single is not None and not single.empty:
                            if 'Close' in single.columns:
                                sc = single[['Close']].copy(); sc.columns = [t]
                                all_close.append(sc)
                            if 'Volume' in single.columns:
                                sv = single[['Volume']].copy(); sv.columns = [t]
                                all_volume.append(sv)
                    except Exception as e2:
                        self._failed_tickers.add(t)
                        if "delisted" in str(e2).lower() or "no price data" in str(e2).lower():
                            self._delisted_tickers.add(t)

            if batch_num < total_batches:
                time.sleep(1.5 + np.random.uniform(0, 1))

        df_close_combined = pd.concat(all_close, axis=1) if all_close else pd.DataFrame()
        df_volume_combined = pd.concat(all_volume, axis=1) if all_volume else pd.DataFrame()
        if not df_close_combined.empty:
            df_close_combined = df_close_combined.loc[:, ~df_close_combined.columns.duplicated()]
        if not df_volume_combined.empty:
            df_volume_combined = df_volume_combined.loc[:, ~df_volume_combined.columns.duplicated()]

        result = {"close": df_close_combined, "volume": df_volume_combined}
        if self.cache and not df_close_combined.empty:
            self.cache.set(cache_key, result, ttl_hours=24)

        logging.info(f"Descarga completada: {len(df_close_combined.columns)} tickers con datos")
        if self._delisted_tickers:
            logging.warning(f"Tickers posiblemente delistados: {len(self._delisted_tickers)}")
        return result

    def get_shares_outstanding(self, tickers):
        shares_map = {}
        for ticker in tickers:
            try:
                t = self._get_ticker(ticker)
                info = t.fast_info
                current_shares = info.get("shares")
                current_price = info.get("lastPrice", 1.0)
                mcap = info.get("marketCap", (current_shares or 1e8) * current_price)
                shares = float(current_shares) if current_shares and current_shares > 0 \
                         else mcap / current_price if current_price > 0 else 1e8
                shares_map[ticker] = shares
            except Exception:
                shares_map[ticker] = 1e8
        return shares_map

    def get_company_info(self, tickers):
        try:
            headers = {"User-Agent": "Mozilla/5.0"}
            resp = self._retry(requests.get, WIKIPEDIA_URL, headers=headers, timeout=20)
            sp_df = pd.read_html(resp.text)[0]
            sp_df = sp_df.rename(columns={
                "Symbol": "Ticker", "Headquarters Location": "HQ",
                "GICS Sector": "Sector", "GICS Sub-Industry": "SubIndustry"
            })
            sp_df["Ticker"] = sp_df["Ticker"].str.replace(".", "-", regex=False)
            return sp_df[["Ticker", "HQ", "Sector", "SubIndustry"]].set_index("Ticker")
        except Exception as e:
            logging.warning(f"Wikipedia fallo: {e}")
            return pd.DataFrame()

    def health_check(self) -> bool:
        try:
            _ = yf.Ticker("AAPL").fast_info.get("lastPrice")
            return True
        except Exception:
            return False

class FreeFloatPolicy:
    STRATEGIC_HOLDING_TYPES = [
        "shares_held_by_officers_directors", "shares_held_by_controlled_entities",
        "shares_held_by_governments", "shares_held_by_private_equity",
        "shares_held_by_strategic_investors", "treasury_stock", "restricted_shares",
    ]
    MIN_FREE_FLOAT_PCT = 0.10

    @staticmethod
    def calculate_free_float_factor(shares_outstanding: float, strategic_holdings: float) -> float:
        if shares_outstanding <= 0:
            return 1.0
        free_float_shares = max(0.0, shares_outstanding - strategic_holdings)
        factor = free_float_shares / shares_outstanding
        return round(factor, 2)

    @staticmethod
    def get_strategic_holdings_proxy(ticker: str, shares_outstanding: float) -> float:
        if shares_outstanding <= 0:
            return 0.0
        KNOWN_FF = {
            'AAPL': 0.99, 'MSFT': 0.99, 'AMZN': 0.88, 'GOOGL': 0.84, 'GOOG': 0.84,
            'META': 0.90, 'TSLA': 0.78, 'BRK-B': 0.99, 'UNH': 0.99, 'JNJ': 0.99,
            'V': 0.99, 'XOM': 0.99, 'WMT': 0.45, 'JPM': 0.99, 'PG': 0.99, 'MA': 0.99,
            'LLY': 0.99, 'HD': 0.99, 'CVX': 0.99, 'MRK': 0.99, 'ABBV': 0.99,
            'PEP': 0.99, 'KO': 0.99, 'BAC': 0.99, 'AVGO': 0.99, 'PFE': 0.99,
            'COST': 0.99, 'TMO': 0.99, 'DIS': 0.99, 'ABT': 0.99, 'ACN': 0.99,
            'WFC': 0.99, 'CSCO': 0.99, 'VZ': 0.99, 'DHR': 0.99, 'LIN': 0.99,
            'TXN': 0.99, 'ADBE': 0.99, 'PM': 0.40, 'NEE': 0.99, 'RTX': 0.99,
            'HON': 0.99, 'CMCSA': 0.99, 'LOW': 0.99, 'IBM': 0.99, 'AMGN': 0.99,
            'UNP': 0.99, 'NKE': 0.89, 'ORCL': 0.45, 'QCOM': 0.99, 'INTU': 0.99,
            'SPGI': 0.99, 'GS': 0.85, 'SBUX': 0.91, 'INTC': 0.99, 'MDT': 0.99,
            'PLD': 0.99, 'BMY': 0.99, 'AMAT': 0.99, 'CAT': 0.99, 'GE': 0.99,
            'T': 0.75, 'DE': 0.85, 'AXP': 0.87, 'MS': 0.85, 'SYK': 0.99,
            'BLK': 0.84, 'C': 0.99, 'MDLZ': 0.99, 'GILD': 0.99, 'ADP': 0.88,
            'CVS': 0.99, 'TJX': 0.99, 'VRTX': 0.99, 'ZTS': 0.99, 'EL': 0.88,
            'PGR': 0.85, 'MO': 0.24, 'CI': 0.99, 'SO': 0.75, 'CB': 0.99,
            'MMC': 0.99, 'DUK': 0.80, 'SHW': 0.90, 'NOC': 0.85, 'BDX': 0.85,
            'CSX': 0.90, 'WM': 0.85, 'PNC': 0.85, 'ISRG': 0.99, 'EOG': 0.95,
            'SLB': 0.99, 'ITW': 0.85, 'FDX': 0.90, 'GD': 0.80, 'USB': 0.90,
            'HUM': 0.95, 'APD': 0.90, 'MRNA': 0.70, 'PYPL': 0.95, 'CME': 0.92,
            'NSC': 0.85, 'GM': 0.75, 'F': 0.30, 'FIS': 0.95, 'ATVI': 0.88,
            'KHC': 0.48, 'KMI': 0.85, 'OXY': 0.85, 'PXD': 0.90, 'MPC': 0.85,
            'PSX': 0.85, 'VLO': 0.90, 'MRO': 0.95, 'DVN': 0.95, 'FANG': 0.90,
            'WMB': 0.90, 'KDP': 0.40, 'CPRT': 0.90, 'ODFL': 0.80, 'FAST': 0.70,
            'LRCX': 0.92, 'KLAC': 0.90, 'SNPS': 0.92, 'CDNS': 0.90, 'ANSS': 0.88,
            'FTNT': 0.85, 'CRWD': 0.88, 'ZS': 0.85, 'OKTA': 0.88, 'DDOG': 0.88,
            'NET': 0.85, 'PLTR': 0.75, 'SNOW': 0.85, 'CRM': 0.92, 'NOW': 0.90,
            'VEEV': 0.90, 'WDAY': 0.88, 'TEAM': 0.85, 'ZM': 0.80, 'DOCU': 0.82,
            'TWLO': 0.85, 'SQ': 0.78, 'SHOP': 0.85, 'SPOT': 0.85, 'UBER': 0.75,
            'LYFT': 0.70, 'ABNB': 0.82, 'DASH': 0.80, 'RBLX': 0.78, 'COIN': 0.72,
            'HOOD': 0.75,
        }
        if ticker in KNOWN_FF:
            ff_factor = KNOWN_FF[ticker]
            return shares_outstanding * (1 - ff_factor)
        return shares_outstanding * 0.20

class DivisorIndexFormula:
    INDEX_LEVEL_DECIMALS = 3
    WEIGHT_DECIMALS = 6
    DIVISOR_DECIMALS = 6

    @staticmethod
    def calculate_index_level(components, prices, shares, free_float, weights, divisor):
        if divisor <= 0:
            return 0.0
        total_float_mcap = 0.0
        for ticker in components:
            price = prices.get(ticker, 0.0)
            share_count = shares.get(ticker, 0.0)
            ff = free_float.get(ticker, 1.0)
            if price > 0 and share_count > 0:
                total_float_mcap += price * share_count * ff
        return round(total_float_mcap / divisor, DivisorIndexFormula.INDEX_LEVEL_DECIMALS)

    @staticmethod
    def adjust_divisor(old_divisor, old_market_cap, new_market_cap):
        if old_market_cap <= 0 or new_market_cap <= 0:
            return old_divisor
        new_divisor = old_divisor * (new_market_cap / old_market_cap)
        return round(new_divisor, DivisorIndexFormula.DIVISOR_DECIMALS)

    @staticmethod
    def initialize_divisor(base_level=1000.0, components=None, prices=None, shares=None, free_float=None):
        if not components or not prices or not shares:
            return base_level
        total_mcap = 0.0
        for ticker in components:
            p = prices.get(ticker, 0.0)
            s = shares.get(ticker, 0.0)
            ff = free_float.get(ticker, 1.0)
            total_mcap += p * s * ff
        if total_mcap <= 0:
            return base_level
        return round(total_mcap / base_level, DivisorIndexFormula.DIVISOR_DECIMALS)

class IndexRulesEngine:
    def __init__(self):
        self.cap_pos = CAP_POSITION
        self.cap_sec = CAP_SECTOR
        self.max_iter = MAX_ITERATIONS
        self.min_adv = MIN_ADV_USD

    def classify_region(self, ticker, as_of_date, base_hq_map):
        if ticker in HQ_CORRECTIONS:
            for start, end, state in HQ_CORRECTIONS[ticker]:
                start_dt = pd.Timestamp(start) if start else pd.Timestamp.min
                end_dt = pd.Timestamp(end) if end else pd.Timestamp.max
                if start_dt <= as_of_date <= end_dt:
                    return Region(MAPA_ESTADOS.get(state, "Neutral"))
        if ticker in SEDES_HISTORICAS_OESTE:
            return Region.WEST
        if ticker in SEDES_HISTORICAS_ESTE:
            return Region.EAST
        return base_hq_map.get(ticker, Region.NEUTRAL)

    def calculate_adv(self, ticker, as_of_date, df_volume, df_close):
        if df_volume.empty or ticker not in df_volume.columns:
            return 0.0
        window_start = as_of_date - pd.Timedelta(days=180)
        vol_window = df_volume[(df_volume.index >= window_start) &
                               (df_volume.index <= as_of_date)][ticker]
        if vol_window.empty or vol_window.isna().all():
            return 0.0
        avg_volume = vol_window.mean()
        price_series = df_close[(df_close.index >= window_start) &
                                (df_close.index <= as_of_date)][ticker]
        price_on_date = price_series.iloc[-1] if not price_series.empty else 0.0
        adv_usd = avg_volume * price_on_date if pd.notna(avg_volume) and pd.notna(price_on_date) else 0.0
        return float(adv_usd)

    def select_components(self, universe, as_of_date, target_region, base_hq_map,
                          df_volume, df_close, shares_map, free_float_map):
        candidates = []
        for ticker in universe:
            region = self.classify_region(ticker, as_of_date, base_hq_map)
            if region != target_region:
                continue
            adv = self.calculate_adv(ticker, as_of_date, df_volume, df_close)
            if adv < self.min_adv:
                continue
            if ticker not in df_close.columns:
                continue
            try:
                price_series = df_close.loc[df_close.index <= as_of_date, ticker]
                if price_series.empty or pd.isna(price_series.iloc[-1]):
                    continue
                price = price_series.iloc[-1]
            except IndexError:
                continue
            shares = shares_map.get(ticker, 1e8)
            ff = free_float_map.get(ticker, 1.0)
            mcap = price * shares * ff
            candidates.append((ticker, mcap))
        candidates.sort(key=lambda x: x[1], reverse=True)
        selected = [ticker for ticker, _ in candidates[:MAX_COMPONENTS]]
        logging.info(f"[{target_region.value}] {len(selected)} components seleccionados "
                     f"(de {len(candidates)} candidatos) en {as_of_date.date()}")
        return selected

    def calculate_weights(self, components, as_of_date, shares_map, free_float_map, sector_map, df_close):
        if not components:
            return {}
        n = len(components)
        weights = {t: 1.0 / n for t in components}
        weights = self._apply_position_cap(weights)
        weights = self._apply_sector_cap(weights, sector_map)
        total = sum(weights.values())
        if total > 0:
            weights = {t: w / total for t, w in weights.items()}
        return weights

    def _apply_position_cap(self, weights):
        weights = dict(weights)
        for _ in range(self.max_iter):
            excess = sum(max(0, w - self.cap_pos) for w in weights.values())
            if excess < 1e-9:
                break
            for t in weights:
                weights[t] = min(weights[t], self.cap_pos)
            eligible = {t: w for t, w in weights.items() if w < self.cap_pos - 1e-9}
            el_total = sum(eligible.values())
            if el_total > 0:
                for t in eligible:
                    weights[t] += (weights[t] / el_total) * excess
        return weights

    def _apply_sector_cap(self, weights, sector_map):
        weights = dict(weights)
        for _ in range(self.max_iter):
            sec_totals = {}
            for t, w in weights.items():
                s = sector_map.get(t, "Unknown")
                sec_totals[s] = sec_totals.get(s, 0) + w
            any_excess = False
            for sec, tot in sec_totals.items():
                if tot > self.cap_sec + 1e-9:
                    any_excess = True
                    scale = self.cap_sec / tot
                    for t in weights:
                        if sector_map.get(t, "Unknown") == sec:
                            weights[t] *= scale
                    exc = tot - self.cap_sec
                    eligible = {t: w for t, w in weights.items()
                                if sector_map.get(t, "Unknown") != sec and w < self.cap_pos - 1e-9}
                    el_total = sum(eligible.values())
                    if el_total > 0:
                        for t in eligible:
                            weights[t] += (weights[t] / el_total) * exc
            if not any_excess:
                break
        return weights

class RebalanceSchedule:
    @staticmethod
    def get_rebalance_dates(start, end):
        dates = []
        start_dt = pd.Timestamp(start)
        end_dt = pd.Timestamp(end)
        for year in range(start_dt.year, end_dt.year + 1):
            for month in REBALANCE_MONTHS:
                reb_date = RebalanceSchedule._third_friday(year, month)
                if start_dt <= reb_date <= end_dt:
                    dates.append(reb_date)
        return sorted(dates)

    @staticmethod
    def _third_friday(year, month):
        first = pd.Timestamp(year=year, month=month, day=1)
        days_until_friday = (4 - first.dayofweek) % 7
        first_friday = first + pd.Timedelta(days=days_until_friday)
        return first_friday + pd.Timedelta(days=14)

    @staticmethod
    def get_selection_date(rebalance_date, trading_index=None):
        if trading_index is not None and len(trading_index) > 0:
            valid_dates = trading_index[trading_index <= rebalance_date]
            if len(valid_dates) >= 6:
                return valid_dates[-6]
        bdates = pd.bdate_range(end=rebalance_date, periods=6, freq='B')
        return bdates[0]

class IndexCalculationEngine:
    def __init__(self, config):
        self.config = config
        self.rules = IndexRulesEngine()
        self.schedule = RebalanceSchedule()

    def calculate_index(self, label, df_close, df_volume, df_bench,
                        hist_df, base_hq_map, shares_map, free_float_map, sector_map):
        """
        FIX v11.3: Level ahora se deriva de los retornos equal-weight acumulados,
        eliminando el desalineamiento con el divisor market-cap.
        """
        logging.info(f"Calculando indice {label}...")
        reb_dates = self.schedule.get_rebalance_dates(self.config.start, self.config.end)

        index_returns = pd.Series(0.0, index=df_close.index)
        holdings_history = OrderedDict()
        turnover_history = OrderedDict()
        prev_weights = {}
        prev_components = []

        for i, reb_date in enumerate(reb_dates):
            selection_date = self.schedule.get_selection_date(reb_date, df_close.index)
            valid_dates = df_close.index[df_close.index <= selection_date]
            if len(valid_dates) == 0:
                continue
            selection_date = valid_dates.max()

            next_date = reb_dates[i + 1] if i + 1 < len(reb_dates) else df_close.index[-1] + pd.Timedelta(days=1)

            universe = self._get_tickers(hist_df, selection_date)
            target_region = Region.WEST

            components = self.rules.select_components(
                universe, selection_date, target_region,
                base_hq_map, df_volume, df_close, shares_map, free_float_map)
            if not components:
                continue

            weights = self.rules.calculate_weights(
                components, selection_date, shares_map, free_float_map, sector_map, df_close)
            if not weights:
                continue

            mask = (df_close.index >= reb_date) & (df_close.index < next_date)
            period_tickers = list(weights.keys())

            period_ret = df_close.loc[mask, period_tickers].pct_change().fillna(0.0)
            w_vec = np.array([weights.get(t, 0) for t in period_ret.columns])
            index_equity = period_ret.dot(w_vec)
            index_returns.loc[mask] = index_equity

            if prev_weights:
                all_t = set(prev_weights.keys()) | set(weights.keys())
                turnover = 0.5 * sum(abs(weights.get(t, 0) - prev_weights.get(t, 0)) for t in all_t)
                turnover_history[reb_date.date()] = turnover

            holdings_history[reb_date.date()] = {t: round(w, 6) for t, w in weights.items()}
            prev_weights = dict(weights)
            prev_components = components

        # FIX CRITICO v11.3: Level = acumulado de retornos equal-weight, base 1000
        index_levels = 1000.0 * (1.0 + index_returns).cumprod()

        metrics = self._calc_index_metrics(index_returns, df_bench)

        return {
            "label": label,
            "metrics": metrics,
            "returns": index_returns,
            "cumulative": (1 + index_returns).cumprod(),
            "index_levels": index_levels,
            "holdings_history": dict(holdings_history),
            "turnover_history": dict(turnover_history),
        }

    def _get_tickers(self, hist_df, fecha):
        fechas = hist_df.index[hist_df.index <= fecha]
        if len(fechas) == 0:
            return []
        raw = str(hist_df.loc[fechas[-1], "tickers"]).split(",")
        clean = []
        for t in raw:
            t = t.strip()
            if t.startswith("$"):
                t = t[1:]
            parts = t.split("-")
            clean.append(parts[0] if len(parts) > 1 and parts[-1].isdigit() else t)
        return list(set(clean))

    def _calc_index_metrics(self, returns, benchmark):
        aligned = pd.concat([returns, benchmark], axis=1).dropna()
        if aligned.empty:
            return {}
        port = aligned.iloc[:, 0]
        bench = aligned.iloc[:, 1]
        port = port[port != 0]
        bench = bench.loc[port.index]
        years = len(port) / TRADING_DAYS_PER_YEAR or 1
        cum = (1 + port).cumprod()

        cagr = (cum.iloc[-1] ** (1 / years) - 1) if cum.iloc[-1] > 0 else 0
        vol = port.std() * np.sqrt(TRADING_DAYS_PER_YEAR)
        rf = self.config.rf_annual
        sharpe = (cagr - rf) / vol if vol > 0 else 0

        neg = port[port < 0]
        dd = neg.std() * np.sqrt(TRADING_DAYS_PER_YEAR) if len(neg) > 0 else 1e-9
        sortino = (cagr - rf) / dd if dd > 0 else 0

        run_max = cum.cummax()
        drawdown_series = (cum - run_max) / run_max
        max_dd = drawdown_series.min()
        calmar = cagr / abs(max_dd) if max_dd != 0 else 0

        up_mask = bench > 0
        down_mask = bench < 0
        upside_capture = (port[up_mask].mean() / bench[up_mask].mean()) * 100 if up_mask.sum() > 0 and bench[up_mask].mean() != 0 else 0
        downside_capture = (port[down_mask].mean() / bench[down_mask].mean()) * 100 if down_mask.sum() > 0 and bench[down_mask].mean() != 0 else 0

        cov = np.cov(port.values, bench.values)[0, 1]
        b_var = bench.var()
        beta = cov / b_var if b_var > 0 else 1

        b_cum = (1 + bench).cumprod()
        b_cagr = (b_cum.iloc[-1] ** (1 / years) - 1) if b_cum.iloc[-1] > 0 else 0
        alpha = cagr - beta * b_cagr

        active = port - bench
        te = active.std() * np.sqrt(TRADING_DAYS_PER_YEAR)
        ir = (cagr - b_cagr) / te if te > 0 else 0

        var_95 = float(np.percentile(port, 5) * 100)
        var_99 = float(np.percentile(port, 1) * 100)
        cvar_95 = float(port[port <= np.percentile(port, 5)].mean() * 100) if len(port[port <= np.percentile(port, 5)]) > 0 else var_95

        skewness = float(port.skew())
        kurt = float(port.kurtosis())
        pos_months = (port > 0).mean() * 100
        gain_pain = abs(port[port > 0].sum() / port[port < 0].sum()) if port[port < 0].sum() != 0 else 0

        gains = port[port > 0].sum()
        losses = abs(port[port < 0].sum())
        omega = gains / losses if losses > 0 else 0

        rolling_window = 63
        rolling_sharpe = (port.rolling(rolling_window).mean() * TRADING_DAYS_PER_YEAR - rf) / (port.rolling(rolling_window).std() * np.sqrt(TRADING_DAYS_PER_YEAR))
        rolling_beta = port.rolling(rolling_window).cov(bench) / bench.rolling(rolling_window).var()

        yearly = port.groupby(port.index.year).apply(lambda x: (1 + x).prod() - 1)

        dd_start = None
        recovery_times = []
        in_drawdown = False
        for date, dd_val in drawdown_series.items():
            if dd_val < -0.01 and not in_drawdown:
                dd_start = date
                in_drawdown = True
            elif dd_val >= -0.005 and in_drawdown and dd_start is not None:
                recovery_times.append((dd_start, date, (date - dd_start).days))
                in_drawdown = False
                dd_start = None

        return {
            "cagr": cagr * 100, "volatility": vol * 100, "sharpe": sharpe,
            "sortino": sortino, "calmar": calmar, "max_drawdown": max_dd * 100,
            "beta": beta, "alpha": alpha * 100, "tracking_error": te * 100,
            "information_ratio": ir, "var_95": var_95, "var_99": var_99,
            "cvar_95": cvar_95, "skewness": skewness, "kurtosis": kurt,
            "positive_months_pct": pos_months, "gain_pain_ratio": gain_pain,
            "omega_ratio": omega, "upside_capture": upside_capture,
            "downside_capture": downside_capture, "total_return": (cum.iloc[-1] - 1) * 100,
            "years": years, "yearly_returns": yearly,
            "drawdown_series": drawdown_series, "recovery_times": recovery_times,
            "rolling_sharpe": rolling_sharpe, "rolling_beta": rolling_beta,
        }

def _newey_west_lag(n: int) -> int:
    return max(1, int(4 * (n / 100) ** (2 / 9)))

def analisis_comparativo(west_returns: pd.Series, qqq_returns: pd.Series,
                          rsp_returns: pd.Series, rf_annual: float) -> Dict[str, Any]:
    result = {}
    for name, bench_returns in [("QQQ", qqq_returns), ("RSP", rsp_returns)]:
        if bench_returns.empty:
            result[name] = None
            continue
        bench_aligned = bench_returns.reindex(west_returns.index).fillna(0)
        mask = (west_returns != 0) & (bench_aligned != 0)
        y = west_returns[mask].values
        x = bench_aligned[mask].values
        n = len(y)
        if n < 30:
            result[name] = None
            continue

        corr = float(np.corrcoef(y, x)[0, 1])
        X = np.column_stack([np.ones_like(x), x])

        beta_hat, _, _, _ = np.linalg.lstsq(X, y, rcond=None)
        alpha_daily, beta = beta_hat[0], beta_hat[1]
        resid = y - X @ beta_hat
        k = X.shape[1]
        sigma2 = (resid @ resid) / (n - k)
        cov_ols = sigma2 * np.linalg.inv(X.T @ X)
        se_alpha_ols = np.sqrt(cov_ols[0, 0])
        t_alpha_ols = alpha_daily / se_alpha_ols if se_alpha_ols > 0 else 0.0
        p_alpha_ols = float(2 * (1 - sp_stats.norm.cdf(abs(t_alpha_ols))))
        ss_res = (resid ** 2).sum()
        ss_tot = ((y - y.mean()) ** 2).sum()
        r_squared = float(1 - ss_res / ss_tot) if ss_tot > 0 else 0.0
        alpha_annual = float(alpha_daily * TRADING_DAYS_PER_YEAR * 100)

        p_alpha_nw = None
        t_alpha_nw = None
        nw_lags = _newey_west_lag(n)
        if HAS_STATSMODELS:
            try:
                model_nw = sm.OLS(y, X).fit(cov_type='HAC', cov_kwds={'maxlags': nw_lags})
                se_alpha_nw = float(model_nw.bse[0])
                t_alpha_nw = float(model_nw.tvalues[0])
                p_alpha_nw = float(model_nw.pvalues[0])
            except Exception as e:
                logging.warning(f"Newey-West fallo para {name}: {e}")

        if p_alpha_nw is None and not HAS_STATSMODELS:
            L = nw_lags
            S = (resid[:, None] * X).T @ (resid[:, None] * X) / n
            for lag in range(1, L + 1):
                w_lag = 1 - lag / (L + 1)
                Gamma = (resid[lag:, None] * X[lag:]).T @ (resid[:-lag, None] * X[:-lag]) / n
                S += w_lag * (Gamma + Gamma.T)
            XtX_inv = np.linalg.inv(X.T @ X)
            cov_nw = n * XtX_inv @ S @ XtX_inv
            se_alpha_nw = np.sqrt(max(cov_nw[0, 0], 0))
            t_alpha_nw = alpha_daily / se_alpha_nw if se_alpha_nw > 0 else 0.0
            p_alpha_nw = float(2 * (1 - sp_stats.norm.cdf(abs(t_alpha_nw))))

        result[name] = {
            "n": n, "correlation": corr, "beta": float(beta), "r_squared": r_squared,
            "alpha_annual_pct": alpha_annual,
            "nw_lags": nw_lags,
            "t_stat_alpha_ols": float(t_alpha_ols), "p_value_alpha_ols": p_alpha_ols,
            "alpha_significant_ols": bool(p_alpha_ols < 0.05),
            "t_stat_alpha_nw": t_alpha_nw, "p_value_alpha_nw": p_alpha_nw,
            "alpha_significant_nw": bool(p_alpha_nw is not None and p_alpha_nw < 0.05),
        }
    return result

# NUEVO v11.3: Diagnostico de Kurtosis
def diagnostico_kurtosis(returns: pd.Series, label: str = "$WEST"):
    """
    Identifica la fuente de kurtosis elevada en los retornos diarios.
    Ejecutar despues de main() para entender que genera colas gruesas.
    """
    rets = returns[returns != 0].copy()
    if len(rets) == 0:
        print("No hay retornos para analizar.")
        return

    print("=" * 70)
    print(f"  DIAGNOSTICO DE KURTOSIS — {label}")
    print("=" * 70)
    print(f"  Observaciones: {len(rets)}")
    print(f"  Kurtosis total: {rets.kurtosis():.2f}")
    print(f"  Skewness: {rets.skew():.2f}")
    print()

    print("  TOP 15 RETORNOS MAS NEGATIVOS:")
    for date, val in rets.nsmallest(15).items():
        print(f"    {date.strftime('%Y-%m-%d')}: {val*100:+.2f}%")
    print()

    print("  TOP 15 RETORNOS MAS POSITIVOS:")
    for date, val in rets.nlargest(15).items():
        print(f"    {date.strftime('%Y-%m-%d')}: {val*100:+.2f}%")
    print()

    eventos = {
        "COVID Crash (Mar 2020)": ("2020-03-01", "2020-03-31"),
        "COVID Rally (Abr 2020)": ("2020-04-01", "2020-04-30"),
        "Elecciones EEUU (Nov 2020)": ("2020-11-01", "2020-11-30"),
        "Inflacion/Rusia (Feb-Mar 2022)": ("2022-02-01", "2022-03-31"),
        "Banking Crisis (Mar 2023)": ("2023-03-01", "2023-03-31"),
    }

    for nombre, (ini, fin) in eventos.items():
        mask = (rets.index >= ini) & (rets.index <= fin)
        subset = rets[mask]
        if len(subset) > 0:
            excluido = rets[~mask]
            print(f"  {nombre}:")
            print(f"    Dias: {len(subset)}, Min: {subset.min()*100:.2f}%, Max: {subset.max()*100:.2f}%")
            print(f"    Kurtosis SIN este periodo: {excluido.kurtosis():.2f}")
            print()

    rets_sin_eventos = rets.copy()
    for _, (ini, fin) in eventos.items():
        rets_sin_eventos = rets_sin_eventos[~((rets_sin_eventos.index >= ini) & (rets_sin_eventos.index <= fin))]

    if rets_sin_eventos.kurtosis() > 8:
        print(f"  ⚠️  Kurtosis sigue alta ({rets_sin_eventos.kurtosis():.2f}) sin eventos conocidos.")
        print("     Posible causa: tickers delistados con gap extremo o error de datos.")
        print("     Revisa fechas aisladas con retornos < -10% o > +10% fuera de los eventos listados.")
    else:
        print(f"  ✅ Kurtosis explicada principalmente por eventos de mercado ({rets_sin_eventos.kurtosis():.2f} sin eventos).")

    print("=" * 70)

class IndexConfig:
    def __init__(self, start, end, rf=0.0415, cache_enabled=True):
        self.start = start
        self.end = end
        self.rf_annual = rf
        self.rf_daily = rf / TRADING_DAYS_PER_YEAR
        self.cache_enabled = cache_enabled

def setup_logging():
    from logging.handlers import RotatingFileHandler
    log_dir = Path("logs"); log_dir.mkdir(exist_ok=True)
    log_file = log_dir / f"mississippi_west_{datetime.now():%Y%m%d}.log"
    logging.basicConfig(
        level=logging.INFO,
        format="%(asctime)s | %(levelname)-8s | %(message)s",
        handlers=[
            RotatingFileHandler(log_file, maxBytes=10_000_000, backupCount=5),
            logging.StreamHandler(sys.stdout)
        ]
    )
    return logging.getLogger(__name__)

def main(start_date=None, end_date=None, no_cache=False):
    logger = setup_logging()
    logger.info("Mississippi Divide West Index v11.3 — Level Fix + Kurtosis Diagnostic")

    in_notebook = False
    try:
        shell = get_ipython().__class__.__name__
        in_notebook = shell == 'ZMQInteractiveShell'
    except NameError:
        pass

    default_start = "2016-01-01"
    default_end = "2024-12-31"

    if not in_notebook:
        try:
            parser = argparse.ArgumentParser()
            parser.add_argument("--start", default=default_start)
            parser.add_argument("--end", default=default_end)
            parser.add_argument("--no-cache", action="store_true")
            args, _ = parser.parse_known_args()
            start = args.start if start_date is None else start_date
            end = args.end if end_date is None else end_date
            use_cache = not args.no_cache if not no_cache else False
        except SystemExit:
            start, end, use_cache = default_start, default_end, True
    else:
        start = start_date or default_start
        end = end_date or default_end
        use_cache = not no_cache

    config = IndexConfig(start=start, end=end, cache_enabled=use_cache)
    logger.info(f"Periodo: {config.start} -> {config.end}")

    cache = CacheManager(Path(".cache") / "mississippi_west.db") if use_cache else None
    provider = YahooProvider(cache_manager=cache)

    if not provider.health_check():
        logger.error("Yahoo Finance no responde"); return 1

    hist_backup_path = Path("data/sp500_historical_backup.csv")
    try:
        hist_df = pd.read_csv(SP500_HIST_URL, index_col=0, parse_dates=True)
        logger.info(f"Historico descargado: {len(hist_df)} registros")
        hist_backup_path.parent.mkdir(parents=True, exist_ok=True)
        hist_df.to_csv(hist_backup_path)
    except Exception as e:
        logger.warning(f"GitHub fallo ({e}), usando backup local...")
        if hist_backup_path.exists():
            hist_df = pd.read_csv(hist_backup_path, index_col=0, parse_dates=True)
        else:
            logger.error("No hay backup local disponible.")
            return 1

    company_info = provider.get_company_info([])
    base_hq_map, sector_map = {}, {}
    for ticker, row in company_info.iterrows():
        region = Region.NEUTRAL
        for part in [p.strip() for p in str(row.get("HQ", "")).split(",")]:
            clean = part.split("[")[0].strip()
            if clean in MAPA_ESTADOS:
                region = Region(MAPA_ESTADOS[clean])
                break
        if region != Region.NEUTRAL:
            base_hq_map[ticker] = region
            sector_map[ticker] = str(row.get("Sector", "Unknown")).strip()

    HISTORICAL_SECTOR_MAP = {
        'AET':'Health Care','AGN':'Health Care','ALXN':'Health Care','ANTM':'Health Care',
        'APC':'Energy','ARNC':'Industrials','ATVI':'Communication Services','BBT':'Financials',
        'BCR':'Health Care','BFB':'Consumer Staples','BHGE':'Energy','BMS':'Materials',
        'BF.B':'Consumer Staples','CA':'Information Technology','CBS':'Communication Services',
        'CELG':'Health Care','CERN':'Health Care','CHTR':'Communication Services','COG':'Energy',
        'COL':'Industrials','COV':'Health Care','CPGX':'Energy','CSRA':'Information Technology',
        'CTXS':'Information Technology','CVS':'Health Care','DELL':'Information Technology',
        'DFS':'Financials','DISCA':'Communication Services','DISCK':'Communication Services',
        'DNB':'Industrials','DO':'Energy','DPS':'Consumer Staples','DRE':'Real Estate',
        'DWDP':'Materials','EMC':'Information Technology','ENDP':'Health Care','EQIX':'Real Estate',
        'ESRX':'Health Care','ETFC':'Financials','EVHC':'Health Care','FDC':'Information Technology',
        'FLIR':'Information Technology','FLS':'Industrials','FTI':'Energy','GAS':'Utilities',
        'GCI':'Communication Services','GGP':'Real Estate','GLW':'Information Technology',
        'GHC':'Communication Services','HAR':'Consumer Discretionary','HCBK':'Financials',
        'HCN':'Real Estate','HCP':'Real Estate','HES':'Energy','HOG':'Consumer Discretionary',
        'HOT':'Real Estate','HRS':'Industrials','HTZ':'Industrials','IBM':'Information Technology',
        'ICE':'Financials','IGT':'Consumer Discretionary','ILMN':'Health Care',
        'INTC':'Information Technology','IPG':'Communication Services','ISRG':'Health Care',
        'ITT':'Industrials','JCP':'Consumer Discretionary','JEC':'Industrials',
        'JNJ':'Health Care','JOY':'Industrials','KORS':'Consumer Discretionary',
        'KRFT':'Consumer Staples','KSS':'Consumer Discretionary','LB':'Consumer Discretionary',
        'LEG':'Consumer Discretionary','LEN':'Consumer Discretionary','LH':'Health Care',
        'LLL':'Industrials','LLTC':'Information Technology','LM':'Financials','LMT':'Industrials',
        'LO':'Consumer Staples','LUK':'Financials','LVLT':'Communication Services',
        'LYB':'Materials','M':'Consumer Discretionary','MAC':'Real Estate',
        'MAR':'Consumer Discretionary','MAS':'Industrials','MAT':'Consumer Discretionary',
        'MBI':'Financials','MCK':'Health Care','MCO':'Financials','MDLZ':'Consumer Staples',
        'MET':'Financials','MHK':'Consumer Discretionary','MJN':'Consumer Staples',
        'MKC':'Consumer Staples','MLM':'Materials','MMC':'Financials','MNK':'Health Care',
        'MON':'Materials','MOS':'Materials','MPC':'Energy','MRK':'Health Care','MRO':'Energy',
        'MS':'Financials','MSFT':'Information Technology','MTB':'Financials',
        'MU':'Information Technology','MUR':'Energy','MYL':'Health Care','NBL':'Energy',
        'NBR':'Energy','NDAQ':'Financials','NEE':'Utilities','NEM':'Materials','NFX':'Energy',
        'NKE':'Consumer Discretionary','NLSN':'Industrials','NOC':'Industrials','NOV':'Energy',
        'NRG':'Utilities','NSC':'Industrials','NTAP':'Information Technology',
        'NTRS':'Financials','NUE':'Materials','NVDA':'Information Technology',
        'NWL':'Consumer Discretionary','NWS':'Communication Services',
        'NWSA':'Communication Services','O':'Real Estate','OXY':'Energy',
        'PAYX':'Information Technology','PBCT':'Financials','PCAR':'Industrials',
        'PCG':'Utilities','PCL':'Real Estate','PCLN':'Consumer Discretionary','PEG':'Utilities',
        'PEP':'Consumer Staples','PFE':'Health Care','PFG':'Financials','PG':'Consumer Staples',
        'PGR':'Financials','PH':'Industrials','PHM':'Consumer Discretionary',
        'PKI':'Health Care','PLD':'Real Estate','PM':'Consumer Staples','PNC':'Financials',
        'PNR':'Industrials','PNW':'Utilities','PPG':'Materials','PPL':'Utilities',
        'PRGO':'Health Care','PRU':'Financials','PSA':'Real Estate','PSX':'Energy',
        'PVH':'Consumer Discretionary','PWR':'Industrials','PX':'Materials','PXD':'Energy',
        'PYPL':'Information Technology','QCOM':'Information Technology','R':'Industrials',
        'RAI':'Consumer Staples','RCL':'Consumer Discretionary','REGN':'Health Care',
        'RF':'Financials','RHI':'Industrials','RHT':'Information Technology','RIG':'Energy',
        'RL':'Consumer Discretionary','ROK':'Industrials','ROP':'Industrials',
        'ROST':'Consumer Discretionary','RRC':'Energy','RSG':'Industrials','RTN':'Industrials',
        'SBUX':'Consumer Discretionary','SCG':'Utilities','SCHW':'Financials','SEE':'Materials',
        'SHW':'Materials','SIAL':'Materials','SIG':'Consumer Discretionary',
        'SJM':'Consumer Staples','SLB':'Energy','SNA':'Industrials',
        'SNDK':'Information Technology','SNI':'Communication Services','SO':'Utilities',
        'SPG':'Real Estate','SPLS':'Consumer Discretionary','SRCL':'Industrials',
        'SRE':'Utilities','STI':'Financials','STJ':'Health Care','STT':'Financials',
        'STX':'Information Technology','STZ':'Consumer Staples','SWK':'Industrials',
        'SWKS':'Information Technology','SWN':'Energy','SYF':'Financials','SYK':'Health Care',
        'SYMC':'Information Technology','SYY':'Consumer Staples','T':'Communication Services',
        'TAP':'Consumer Staples','TDC':'Information Technology','TDG':'Industrials',
        'TE':'Utilities','TEG':'Utilities','TEL':'Information Technology',
        'TGNA':'Communication Services','TGT':'Consumer Discretionary','THC':'Health Care',
        'TIF':'Consumer Discretionary','TJX':'Consumer Discretionary','TMK':'Financials',
        'TMO':'Health Care','TRIP':'Communication Services','TROW':'Financials',
        'TRV':'Financials','TSCO':'Consumer Discretionary','TSN':'Consumer Staples',
        'TSO':'Energy','TSS':'Information Technology','TWX':'Communication Services',
        'TXN':'Information Technology','TXT':'Industrials','TYC':'Industrials',
        'UA':'Consumer Discretionary','UAA':'Consumer Discretionary','UAL':'Industrials',
        'UDR':'Real Estate','UHS':'Health Care','UNH':'Health Care','UNM':'Financials',
        'UNP':'Industrials','UPS':'Industrials','URBN':'Consumer Discretionary',
        'URI':'Industrials','USB':'Financials','UTX':'Industrials',
        'V':'Information Technology','VAR':'Health Care','VFC':'Consumer Discretionary',
        'VIAB':'Communication Services','VLO':'Energy','VMC':'Materials','VNO':'Real Estate',
        'VRSK':'Industrials','VRSN':'Information Technology','VRTX':'Health Care',
        'VTR':'Real Estate','VZ':'Communication Services','WAT':'Health Care',
        'WBA':'Consumer Staples','WDC':'Information Technology','WEC':'Utilities',
        'WFC':'Financials','WFM':'Consumer Staples','WHR':'Consumer Discretionary',
        'WM':'Industrials','WMB':'Energy','WMT':'Consumer Staples','WRK':'Materials',
        'WU':'Information Technology','WY':'Real Estate','WYN':'Consumer Discretionary',
        'WYNN':'Consumer Discretionary','XEC':'Energy','XEL':'Utilities','XL':'Financials',
        'XLNX':'Information Technology','XOM':'Energy','XRAY':'Health Care',
        'XRX':'Information Technology','XYL':'Industrials','YHOO':'Communication Services',
        'YUM':'Consumer Discretionary','ZBH':'Health Care','ZTS':'Health Care',
    }
    for ticker, sector in HISTORICAL_SECTOR_MAP.items():
        if ticker not in sector_map:
            sector_map[ticker] = sector

    logger.info(f"Clasificadas: {len(base_hq_map)} empresas | Sectores: {len(sector_map)}")

    all_tickers = set()
    for fecha in pd.date_range(start=config.start, end=config.end, freq="QS"):
        fechas = hist_df.index[hist_df.index <= fecha]
        if len(fechas) > 0:
            raw = str(hist_df.loc[fechas[-1], "tickers"]).split(",")
            for t in raw:
                t = t.strip()
                if t.startswith("$"):
                    t = t[1:]
                parts = t.split("-")
                all_tickers.add(parts[0] if len(parts) > 1 and parts[-1].isdigit() else t)
    all_tickers |= {"^GSPC", "QQQ", "RSP"}
    logger.info(f"Universo: {len(all_tickers)} tickers")

    download_start = datetime.strptime(config.start, "%Y-%m-%d") - timedelta(days=365)
    download_end = datetime.strptime(config.end, "%Y-%m-%d")
    price_data = provider.get_historical_prices(list(all_tickers), download_start, download_end)
    df_full = price_data["close"]
    df_vol = price_data.get("volume", pd.DataFrame())
    backtest_start = pd.Timestamp(config.start)
    backtest_end = pd.Timestamp(config.end)
    df_close = df_full[(df_full.index >= backtest_start) & (df_full.index <= backtest_end)]

    if "^GSPC" not in df_close.columns:
        logger.error("Falta ^GSPC"); return 1

    df_bench = df_close["^GSPC"].pct_change().fillna(0.0)
    df_qqq_ret = df_close["QQQ"].pct_change().fillna(0.0) if "QQQ" in df_close.columns else pd.Series()
    df_rsp_ret = df_close["RSP"].pct_change().fillna(0.0) if "RSP" in df_close.columns else pd.Series()

    logger.info("Descargando shares outstanding (proxy)...")
    shares_map = provider.get_shares_outstanding(list(all_tickers - {"^GSPC", "QQQ", "RSP"}))
    logger.warning("Shares actuales usados como proxy. En produccion: Compustat/CRSP.")

    logger.info("Calculando free float factors...")
    free_float_map = {}
    for ticker in all_tickers - {"^GSPC", "QQQ", "RSP"}:
        shares = shares_map.get(ticker, 1e8)
        strategic = FreeFloatPolicy.get_strategic_holdings_proxy(ticker, shares)
        ff = FreeFloatPolicy.calculate_free_float_factor(shares, strategic)
        free_float_map[ticker] = ff
    logger.info(f"Free float calculado para {len(free_float_map)} tickers")

    engine = IndexCalculationEngine(config)
    res_west = engine.calculate_index(Side.WEST, df_close, df_vol, df_bench,
                                       hist_df, base_hq_map, shares_map, free_float_map, sector_map)

    def bench_stats(ticker):
        if ticker not in df_close.columns:
            return None
        r = df_close[ticker].pct_change().fillna(0)
        yrs = len(r) / 252 or 1
        cum = (1 + r).cumprod()
        cagr = (cum.iloc[-1] ** (1 / yrs) - 1) * 100
        vol = r.std() * np.sqrt(252) * 100
        sh = (cagr - config.rf_annual * 100) / vol if vol > 0 else 0
        dd = ((cum - cum.cummax()) / cum.cummax()).min() * 100
        neg = r[r < 0]
        sort = (cagr/100 - config.rf_annual) / (neg.std() * np.sqrt(252)) if len(neg) > 0 and neg.std() > 0 else 0
        var95 = float(np.percentile(r, 5) * 100)
        var99 = float(np.percentile(r, 1) * 100)
        return {"cagr": cagr, "volatility": vol, "sharpe": sh, "sortino": sort,
                "max_drawdown": dd, "var_95": var95, "var_99": var99,
                "total_return": (cum.iloc[-1] - 1) * 100}

    sp500_m = bench_stats("^GSPC")
    qqq_m = bench_stats("QQQ")
    rsp_m = bench_stats("RSP")

    comparativo = analisis_comparativo(res_west["returns"], df_qqq_ret, df_rsp_ret, config.rf_annual)

    w = res_west["metrics"]
    print("\n" + "=" * 110)
    print("  MISSISSIPPI DIVIDE WEST INDEX v11.3 — INVESTOR PRESENTATION EDITION (EQUAL WEIGHT)")
    print("=" * 110)
    print(f"  Periodo: {config.start} -> {config.end}  |  Base Level: 1,000  |  Equal Weight (1/N)")
    print("-" * 110)

    print(f"\n  {'METRICA':<28} {'WEST':>14} {'S&P 500':>14} {'QQQ':>14} {'RSP (EW)':>14}")
    print("  " + "-" * 78)
    metrics_rows = [
        ("CAGR", f"{w['cagr']:>13.2f}%", f"{sp500_m['cagr']:>13.2f}%", f"{qqq_m['cagr']:>13.2f}%", f"{rsp_m['cagr']:>13.2f}%"),
        ("Volatilidad Anual", f"{w['volatility']:>13.2f}%", f"{sp500_m['volatility']:>13.2f}%", f"{qqq_m['volatility']:>13.2f}%", f"{rsp_m['volatility']:>13.2f}%"),
        ("Sharpe Ratio", f"{w['sharpe']:>14.2f}", f"{sp500_m['sharpe']:>14.2f}", f"{qqq_m['sharpe']:>14.2f}", f"{rsp_m['sharpe']:>14.2f}"),
        ("Sortino Ratio", f"{w['sortino']:>14.2f}", f"{sp500_m['sortino']:>14.2f}", f"{qqq_m['sortino']:>14.2f}", f"{rsp_m['sortino']:>14.2f}"),
        ("Calmar Ratio", f"{w['calmar']:>14.2f}", "—", "—", "—"),
        ("Max Drawdown", f"{w['max_drawdown']:>13.2f}%", f"{sp500_m['max_drawdown']:>13.2f}%", f"{qqq_m['max_drawdown']:>13.2f}%", f"{rsp_m['max_drawdown']:>13.2f}%"),
        ("Alpha vs S&P 500", f"{w['alpha']:>13.2f}%", "—", "—", "—"),
        ("Beta vs S&P 500", f"{w['beta']:>14.2f}", "1.00", "—", "—"),
        ("Tracking Error", f"{w['tracking_error']:>13.2f}%", "—", "—", "—"),
        ("Information Ratio", f"{w['information_ratio']:>14.2f}", "—", "—", "—"),
        ("Upside Capture", f"{w['upside_capture']:>13.1f}%", "—", "—", "—"),
        ("Downside Capture", f"{w['downside_capture']:>13.1f}%", "—", "—", "—"),
        ("VaR 95% (daily)", f"{w['var_95']:>13.2f}%", f"{sp500_m['var_95']:>13.2f}%", f"{qqq_m['var_95']:>13.2f}%", f"{rsp_m['var_95']:>13.2f}%"),
        ("CVaR 95% (daily)", f"{w['cvar_95']:>13.2f}%", "—", "—", "—"),
        ("Skewness", f"{w['skewness']:>14.2f}", "—", "—", "—"),
        ("Kurtosis", f"{w['kurtosis']:>14.2f}", "—", "—", "—"),
        ("Positive Months %", f"{w['positive_months_pct']:>13.1f}%", "—", "—", "—"),
        ("Gain/Pain Ratio", f"{w['gain_pain_ratio']:>14.2f}", "—", "—", "—"),
        ("Omega Ratio", f"{w['omega_ratio']:>14.2f}", "—", "—", "—"),
        ("Total Return", f"{w['total_return']:>13.2f}%", f"{sp500_m['total_return']:>13.2f}%", f"{qqq_m['total_return']:>13.2f}%", f"{rsp_m['total_return']:>13.2f}%"),
    ]
    for row in metrics_rows:
        print(f"  {row[0]:<28} {row[1]:>14} {row[2]:>14} {row[3]:>14} {row[4]:>14}")

    print("\n" + "=" * 70)
    print("  ¿$WEST ES DIFERENTE DE QQQ Y RSP? (correlacion + regresion de alpha)")
    print("=" * 70)
    for name in ["QQQ", "RSP"]:
        comp = comparativo.get(name)
        if comp is None:
            print(f"\n  vs {name}: no disponible")
            continue
        print(f"\n  vs {name}:  (Newey-West lags={comp['nw_lags']})")
        print(f"    Correlacion:  {comp['correlation']:.4f}")
        print(f"    Beta:         {comp['beta']:.3f}")
        print(f"    R²:           {comp['r_squared']:.3f}")
        print(f"    Alpha anual:  {comp['alpha_annual_pct']:+.2f}%")
        print(f"    p-value OLS simple:    {comp['p_value_alpha_ols']:.4f}")
        if comp['p_value_alpha_nw'] is not None:
            print(f"    p-value Newey-West:    {comp['p_value_alpha_nw']:.4f}  <- EL QUE IMPORTA")
        else:
            print(f"    p-value Newey-West:    no disponible")
        if comp["alpha_significant_nw"]:
            print(f"    ✅ Alpha SIGNIFICATIVO incluso con Newey-West — evidencia solida de edge real")
        elif comp["alpha_significant_ols"]:
            print(f"    ⚠️  Alpha significativo en OLS simple PERO NO en Newey-West — la")
            print(f"       autocorrelacion estaba inflando la confianza. Tratar con cautela.")
        else:
            print(f"    ⚠️  Alpha NO significativo en ninguna de las dos pruebas.")

    bench_years = df_bench.groupby(df_bench.index.year).apply(lambda x: (1 + x).prod() - 1)
    qqq_years = df_qqq_ret.groupby(df_qqq_ret.index.year).apply(lambda x: (1 + x).prod() - 1) if not df_qqq_ret.empty else pd.Series()
    rsp_years = df_rsp_ret.groupby(df_rsp_ret.index.year).apply(lambda x: (1 + x).prod() - 1) if not df_rsp_ret.empty else pd.Series()

    print(f"\n  === MARCADOR ANO A ANO ===")
    print(f"  {'Ano':<6} {'WEST':>10} {'S&P 500':>10} {'QQQ':>10} {'RSP':>10}  Ganador")
    print("  " + "-" * 60)
    for yr in sorted(w["yearly_returns"].index):
        west_yr = w["yearly_returns"].get(yr, 0)
        sp_yr = bench_years.get(yr, 0)
        qq_yr = qqq_years.get(yr, 0) if yr in qqq_years.index else float('nan')
        rs_yr = rsp_years.get(yr, 0) if yr in rsp_years.index else float('nan')
        winner = "WEST" if west_yr > sp_yr else "S&P"
        print(f"  {yr:<6} {west_yr * 100:>+9.1f}% {sp_yr * 100:>+9.1f}% {qq_yr * 100:>+9.1f}% {rs_yr * 100:>+9.1f}%  {winner}")
    print("=" * 110)

    avg_turnover = 0.0
    max_turnover = 0.0
    if res_west["turnover_history"]:
        avg_turnover = float(np.mean(list(res_west["turnover_history"].values())))
        max_turnover = float(max(res_west["turnover_history"].values()))
        print(f"\n  TURNOVER ANALYSIS")
        print(f"  Promedio por rebalancing: {avg_turnover*100:.2f}%")
        print(f"  Maximo por rebalancing:   {max_turnover*100:.2f}%")
        print(f"  Rebalancings ejecutados: {len(res_west['turnover_history'])}")

    if w.get("recovery_times"):
        print(f"\n  DRAWDOWN RECOVERY ANALYSIS")
        print(f"  Numero de drawdowns >1%: {len(w['recovery_times'])}")
        avg_recovery = np.mean([rt[2] for rt in w['recovery_times']])
        print(f"  Tiempo promedio de recuperacion: {avg_recovery:.0f} dias")

    # NUEVO v11.3: Diagnostico de Kurtosis automatico
    print("\n")
    diagnostico_kurtosis(res_west["returns"], label="$WEST")

    output = Path("output"); output.mkdir(exist_ok=True)
    pd.DataFrame.from_dict(res_west["holdings_history"], orient="index").to_csv(
        output / "WEST_composition.csv")
    pd.DataFrame({
        "WEST_Return": res_west["returns"], "WEST_Cum": res_west["cumulative"],
        "SP500_Return": df_bench, "SP500_Cum": (1 + df_bench).cumprod()
    }).to_csv(output / "index_performance.csv")

    # Guardar niveles del indice para referencia
    res_west["index_levels"].to_csv(output / "WEST_index_levels.csv")

    # GRAFICO
    cum_w = res_west["cumulative"]
    cum_s = (1 + df_bench).cumprod()
    cum_q = (1 + df_qqq_ret).cumprod() if not df_qqq_ret.empty else None
    cum_r = (1 + df_rsp_ret).cumprod() if not df_rsp_ret.empty else None

    fig, axes = plt.subplots(2, 2, figsize=(20, 14))
    fig.patch.set_facecolor('#0B0E17')
    fig.suptitle('Mississippi Divide West Index v11.3 — Investor Presentation (Equal Weight)',
                 fontsize=18, color='white', fontweight='bold', y=0.98)

    ax1 = axes[0, 0]
    ax1.set_facecolor('#0B0E17')
    ax1.plot(cum_w, color='#FF8C42', linewidth=2.8, label='$WEST')
    ax1.plot(cum_s, color='#8B8B8B', linewidth=1.8, linestyle='--', label='S&P 500')
    if cum_q is not None:
        ax1.plot(cum_q, color='#A855F7', linewidth=1.4, linestyle=':', label='QQQ')
    if cum_r is not None:
        ax1.plot(cum_r, color='#22C55E', linewidth=1.4, linestyle=':', label='RSP (EW)')
    ax1.fill_between(cum_w.index, 1, cum_w, alpha=0.08, color='#FF8C42')
    last = cum_w.index[-1]
    for cum, color, lbl in [(cum_w, '#FF8C42', '$WEST'), (cum_s, '#8B8B8B', 'S&P')]:
        ax1.annotate(f"{cum.iloc[-1]:.2f}x", xy=(last, cum.iloc[-1]),
                    xytext=(8, 0), textcoords='offset points',
                    color=color, fontsize=11, fontweight='bold', va='center')
    ax1.set_title('Cumulative Performance (Base = 1)', color='white', fontsize=12)
    ax1.tick_params(colors='#AAAAAA')
    for spine in ax1.spines.values():
        spine.set_edgecolor('#333333')
    ax1.grid(True, alpha=0.15, color='#444444')
    ax1.legend(fontsize=10, facecolor='#1A1D2E', edgecolor='#333355',
              labelcolor='white', loc='upper left')

    ax2 = axes[0, 1]
    ax2.set_facecolor('#0B0E17')
    dd_w = w["drawdown_series"] * 100
    dd_s = ((cum_s - cum_s.cummax()) / cum_s.cummax()) * 100
    ax2.fill_between(dd_w.index, dd_w, 0, alpha=0.25, color='#FF8C42', label='$WEST')
    ax2.fill_between(dd_s.index, dd_s, 0, alpha=0.15, color='#8B8B8B', label='S&P 500')
    ax2.axhline(y=-20, color='red', linestyle=':', alpha=0.5, linewidth=1)
    ax2.set_title('Drawdown Analysis (%)', color='white', fontsize=12)
    ax2.tick_params(colors='#AAAAAA')
    for spine in ax2.spines.values():
        spine.set_edgecolor('#333333')
    ax2.grid(True, alpha=0.15, color='#444444')
    ax2.legend(fontsize=10, facecolor='#1A1D2E', edgecolor='#333355',
              labelcolor='white', loc='lower left')

    ax3 = axes[1, 0]
    ax3.set_facecolor('#0B0E17')
    rs_w = w["rolling_sharpe"]
    rs_s = ((df_bench.rolling(63).mean() * 252 - config.rf_annual) /
            (df_bench.rolling(63).std() * np.sqrt(252)))
    ax3.plot(rs_w, color='#FF8C42', linewidth=1.5, label='$WEST')
    ax3.plot(rs_s, color='#8B8B8B', linewidth=1.2, linestyle='--', label='S&P 500')
    ax3.axhline(y=0, color='white', alpha=0.3, linewidth=0.5)
    ax3.set_title('Rolling 63-Day Sharpe Ratio', color='white', fontsize=12)
    ax3.tick_params(colors='#AAAAAA')
    for spine in ax3.spines.values():
        spine.set_edgecolor('#333333')
    ax3.grid(True, alpha=0.15, color='#444444')
    ax3.legend(fontsize=10, facecolor='#1A1D2E', edgecolor='#333355',
              labelcolor='white', loc='upper left')

    ax4 = axes[1, 1]
    ax4.set_facecolor('#0B0E17')
    rb_w = w["rolling_beta"]
    ax4.plot(rb_w, color='#FF8C42', linewidth=1.5, label='$WEST vs S&P 500')
    ax4.axhline(y=1.0, color='white', alpha=0.3, linewidth=0.5, linestyle='--')
    ax4.set_title('Rolling 63-Day Beta vs S&P 500', color='white', fontsize=12)
    ax4.tick_params(colors='#AAAAAA')
    for spine in ax4.spines.values():
        spine.set_edgecolor('#333333')
    ax4.grid(True, alpha=0.15, color='#444444')
    ax4.legend(fontsize=10, facecolor='#1A1D2E', edgecolor='#333355',
              labelcolor='white', loc='upper left')

    plt.tight_layout(rect=[0, 0, 1, 0.96])
    plt.savefig(output / "Mississippi_West_v11_PitchDeck.png",
                dpi=300, bbox_inches='tight', facecolor=fig.get_facecolor())
    plt.show()

    # REPORTE MARKDOWN
    qqq_comp = comparativo.get("QQQ")
    rsp_comp = comparativo.get("RSP")

    def comp_block(name, comp):
        if comp is None:
            return f"### vs {name}\n\nNo disponible.\n"
        p_nw_str = f"{comp['p_value_alpha_nw']:.4f}" if comp['p_value_alpha_nw'] is not None else "N/A"
        if comp["alpha_significant_nw"]:
            veredicto = "SI, incluso con Newey-West (HAC) — evidencia solida"
        elif comp["alpha_significant_ols"]:
            veredicto = "Significativo en OLS simple pero NO en Newey-West — probablemente inflado por autocorrelacion, tratar con cautela"
        else:
            veredicto = "NO CONFIRMADO en ninguna de las dos pruebas"
        return f"""### vs {name}

- Correlacion diaria: {comp['correlation']:.4f}
- Beta: {comp['beta']:.3f}
- R²: {comp['r_squared']:.3f}
- Alpha anualizado: {comp['alpha_annual_pct']:+.2f}%
- p-value (OLS simple): {comp['p_value_alpha_ols']:.4f}
- p-value (Newey-West, {comp['nw_lags']} lags): {p_nw_str}
- **Veredicto: {veredicto}**
"""

    report_md = f"""# Mississippi Divide West Index v11.3
## Executive Summary for Investor Presentation (Equal Weight, con analisis vs RSP/QQQ)

**Period:** {config.start} to {config.end}
**Weighting:** Equal Weight (1/N) — cambiado desde float-adjusted market cap
por concentracion excesiva en megacaps tech (ver seccion de limitaciones).
**Rebalancing:** Quarterly (3rd Friday of Mar/Jun/Sep/Dec)
**Universe:** S&P 500 constituents with HQ West of the Mississippi River

---

## 0. Limitaciones conocidas (leer primero)

1. Shares outstanding: proxy actual, no point-in-time historico.
2. Free float: estimado, no medido con datos institucionales.
3. p-values de la seccion "Es diferente de QQQ/RSP?" usan OLS simple sin
   correccion de autocorrelacion (Newey-West) — tratar como indicativo.
4. Metodologia iterada ~11 veces buscando resultados favorables — riesgo
   de overfitting no descartado sin validacion fuera de muestra (2025-26).
5. **FIX v11.3:** El Level del indice ahora es consistente con los retornos
   equal-weight (anteriormente usaba divisor market-cap, generando desalineamiento).

---

## 1. Performance Metrics

| Metric | $WEST | S&P 500 | QQQ | RSP (EW) |
|--------|-------|---------|-----|----------|
| **CAGR** | {w['cagr']:.2f}% | {sp500_m['cagr']:.2f}% | {qqq_m['cagr']:.2f}% | {rsp_m['cagr']:.2f}% |
| **Volatility** | {w['volatility']:.2f}% | {sp500_m['volatility']:.2f}% | {qqq_m['volatility']:.2f}% | {rsp_m['volatility']:.2f}% |
| **Sharpe** | {w['sharpe']:.2f} | {sp500_m['sharpe']:.2f} | {qqq_m['sharpe']:.2f} | {rsp_m['sharpe']:.2f} |
| **Sortino** | {w['sortino']:.2f} | {sp500_m['sortino']:.2f} | {qqq_m['sortino']:.2f} | {rsp_m['sortino']:.2f} |
| **Max Drawdown** | {w['max_drawdown']:.2f}% | {sp500_m['max_drawdown']:.2f}% | {qqq_m['max_drawdown']:.2f}% | {rsp_m['max_drawdown']:.2f}% |
| **Omega Ratio** | {w['omega_ratio']:.2f} | — | — | — |
| **CVaR 95%** | {w['cvar_95']:.2f}% | — | — | — |
| **Kurtosis** | {w['kurtosis']:.2f} | — | — | — |

## 2. ¿$WEST es diferente de QQQ y RSP? (lo que decide todo)

{comp_block("QQQ", qqq_comp)}

{comp_block("RSP", rsp_comp)}

## 3. Turnover & Drawdown Recovery

- Turnover promedio por rebalanceo: {avg_turnover*100:.2f}%
- Turnover maximo: {max_turnover*100:.2f}%
- Drawdowns >1%: {len(w.get('recovery_times', []))}
- Tiempo promedio de recuperacion: {np.mean([rt[2] for rt in w.get('recovery_times', [])]) if w.get('recovery_times') else float('nan'):.0f} dias

---

*Disclaimer: Backtest preliminar para discusion con index providers.
No apto para distribucion final a inversionistas sin resolver la seccion 0.*
"""

    with open(output / "executive_report.md", "w", encoding="utf-8") as f:
        f.write(report_md)

    print(f"\n  Resultados guardados en: {output}/")
    print("   • WEST_composition.csv | index_performance.csv | WEST_index_levels.csv")
    print("   • Mississippi_West_v11_PitchDeck.png | executive_report.md")
    print("\n  ✅ v11.3: Level del indice ahora es consistente con retornos equal-weight.")
    logger.info("Index calculation v11.3 completado")
    return 0


if __name__ == "__main__":
    try:
        main()
    except SystemExit:
        pass
