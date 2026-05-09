import { lazy, Suspense, useEffect, useMemo, useRef, useState } from "react";
import {
  Target, Settings, X, Flame, Bell, Plus, Minus, Home, Newspaper,
  TrendingUp, HelpCircle, Trash2, Trophy, ExternalLink,
  RefreshCw, Pencil, Check, Calendar, ChevronLeft, ChevronRight, Activity,
} from "lucide-react";

const EvolutionTab = lazy(() => import("./EvolutionTab"));
import { Button } from "@/components/ui/button";
import { Input } from "@/components/ui/input";
import { toast } from "@/hooks/use-toast";
import { CurrencyInput } from "@/components/CurrencyInput";
import {
  Carousel, CarouselContent, CarouselItem,
  type CarouselApi,
} from "@/components/ui/carousel";

// Limite máximo de metas ativas
const MAX_GOALS = 10;
import { Sparkline } from "@/components/Sparkline";
import { AssetChart } from "@/components/AssetChart";
import { ErrorBoundary } from "@/components/ErrorBoundary";
import { Confetti } from "@/components/Confetti";
import {
  useGoalStore, formatBRL, defaultDeadline, todayISO,
  goalSaved, goalProgress, goalPerDay, daysUntil,
  CATEGORIES, CATEGORY_ICONS, CATEGORY_COLORS,
  type Goal, type Category,
} from "@/hooks/useGoalStore";
import {
  fetchNews, timeAgo, getSeenIds, markSeen, type NewsItem,
} from "@/lib/news";
import {
  getRelevantNews, getRateAlerts, type ExchangeRate, type RateAlert,
} from "@/lib/notifications";
import {
  useTheme, THEME_PRESETS, MOOD_PRESETS,
  getActivePresetId, setActivePresetId,
  wasSuggestionDismissed, dismissSuggestion,
  type MoodPresetId,
} from "@/hooks/useTheme";
import { ColorWheel } from "@/components/ColorWheel";
import { Slider } from "@/components/ui/slider";

type Tab = "home" | "news" | "investments" | "evolution" | "help";

// ─── GOAL CARD ────────────────────────────────────────────────────────────────

const GoalCard = ({
  goal, onDeposit, onWithdraw, onRemove,
}: {
  goal: Goal;
  onDeposit: (goalId: string, amount: number) => void;
  onWithdraw: (goalId: string, amount: number) => void;
  onRemove: (id: string) => void;
}) => {
  const [quickMode, setQuickMode] = useState<null | "add" | "sub">(null);
  const [quickAmount, setQuickAmount] = useState<number>(0);
  const color = CATEGORY_COLORS[goal.category];
  const saved = goalSaved(goal);
  const progress = goalProgress(goal);
  const perDay = goalPerDay(goal);
  const days = daysUntil(goal.deadline);
  const done = progress >= 100;

  const handleQuick = () => {
    if (quickAmount <= 0 || !quickMode) return;
    if (quickMode === "add") onDeposit(goal.id, quickAmount);
    else onWithdraw(goal.id, Math.min(quickAmount, saved));
    setQuickAmount(0);
    setQuickMode(null);
  };

  return (
    <div className="bg-card/50 border border-border/50 rounded-2xl p-5 hover:border-border transition-all group relative overflow-hidden"
         style={{ borderLeftColor: color, borderLeftWidth: 3 }}>
      {/* Header */}
      <div className="flex justify-between items-start mb-3">
        <div className="flex items-center gap-2.5 min-w-0 flex-1">
          <span className="text-xl shrink-0">{CATEGORY_ICONS[goal.category]}</span>
          <div className="min-w-0">
            <h3 className="font-semibold text-base truncate">{goal.item}</h3>
            <p className="text-xs text-muted-foreground mt-0.5">
              {goal.category} · {days} dias restantes
            </p>
          </div>
        </div>
        <div className="flex items-center gap-1 shrink-0 ml-2">
          <button
            onClick={() => setQuickMode((m) => (m === "add" ? null : "add"))}
            aria-label="Depósito rápido"
            className="p-1.5 rounded-lg bg-primary/15 border border-primary/30 text-primary hover:bg-primary/25 transition-colors"
          >
            <Plus className="w-4 h-4" />
          </button>
          <button
            onClick={() => setQuickMode((m) => (m === "sub" ? null : "sub"))}
            aria-label="Estorno"
            disabled={saved <= 0}
            className="p-1.5 rounded-lg bg-destructive/10 border border-destructive/30 text-destructive hover:bg-destructive/20 transition-colors disabled:opacity-40"
          >
            <Minus className="w-4 h-4" />
          </button>
          <button
            onClick={() => onRemove(goal.id)}
            aria-label="Remover meta"
            className="p-1.5 opacity-0 group-hover:opacity-100 hover:bg-destructive/20 rounded-lg transition-all"
          >
            <X className="w-4 h-4 text-destructive" />
          </button>
        </div>
      </div>

      {/* Quick deposit/withdraw popover (inline) */}
      {quickMode && (
        <div className="mb-3 flex gap-2 animate-fade-in">
          <CurrencyInput
            value={quickAmount}
            onChange={setQuickAmount}
            placeholder={quickMode === "add" ? "Depositar R$" : "Estornar R$"}
            autoFocus
            onKeyDown={(e) => e.key === "Enter" && handleQuick()}
            className="h-9 rounded-lg bg-secondary/60 border-border/40 text-sm"
          />
          <Button
            size="sm"
            disabled={quickAmount <= 0}
            onClick={handleQuick}
            className={`h-9 rounded-lg text-primary-foreground disabled:opacity-50 ${
              quickMode === "add" ? "bg-gradient-primary" : "bg-destructive hover:bg-destructive/90"
            }`}
          >
            <Check className="w-4 h-4" />
          </Button>
        </div>
      )}

      {/* Progress bar */}
      <div className="w-full bg-secondary/60 rounded-full h-1.5 mb-3 overflow-hidden">
        <div
          className="h-full rounded-full transition-all duration-500"
          style={{ width: `${progress}%`, backgroundColor: color }}
        />
      </div>

      {/* Sparkline */}
      <div className="mb-3 -mx-1">
        <Sparkline goal={goal} color={color} />
      </div>

      {/* Stats */}
      <div className="grid grid-cols-3 gap-2 text-xs mb-3">
        <div className="bg-secondary/40 rounded-lg p-2.5">
          <p className="text-muted-foreground text-[10px] mb-0.5">Poupado</p>
          <p className="font-semibold">{formatBRL(saved)}</p>
        </div>
        <div className="bg-secondary/40 rounded-lg p-2.5">
          <p className="text-muted-foreground text-[10px] mb-0.5">Meta</p>
          <p className="font-semibold">{formatBRL(goal.total)}</p>
        </div>
        <div className="bg-secondary/40 rounded-lg p-2.5">
          <p className="text-muted-foreground text-[10px] mb-0.5">Progresso</p>
          <p className="font-semibold" style={{ color: done ? color : undefined }}>
            {done ? "🏆" : `${Math.round(progress)}%`}
          </p>
        </div>
      </div>

      {/* Daily target */}
      {!done && (
        <div className="bg-secondary/30 border border-border/30 rounded-lg p-2.5 text-xs mb-3">
          <p className="text-muted-foreground">
            Para bater a meta, poupe{" "}
            <span className="font-semibold" style={{ color }}>
              {formatBRL(perDay)}/dia
            </span>
          </p>
        </div>
      )}

      {/* Preset quick adds */}
      <div className="flex gap-2">
        {[50, 100, 200].map((amount) => (
          <button
            key={amount}
            onClick={() => onDeposit(goal.id, amount)}
            className="flex-1 py-2 rounded-lg text-xs font-semibold transition-all bg-primary/10 border border-primary/25 text-primary hover:bg-primary/20"
          >
            +R${amount}
          </button>
        ))}
      </div>
    </div>
  );
};

// ─── NEW GOAL MODAL ──────────────────────────────────────────────────────────

const NewGoalModal = ({
  onClose, onAdd,
}: {
  onClose: () => void;
  onAdd: (item: string, total: number, deadline: string, category: Category) => void;
}) => {
  const [item, setItem] = useState("");
  const [total, setTotal] = useState<number>(0);
  const [deadline, setDeadline] = useState(defaultDeadline());
  const [category, setCategory] = useState<Category>("Outros");
  const valid = item.trim().length > 0 && total > 0 && !!deadline;

  return (
    <div className="fixed inset-0 bg-black/80 flex items-center justify-center z-50 p-4 backdrop-blur-sm">
      <div className="bg-card border border-border rounded-3xl max-w-md w-full p-6 animate-scale-in">
        <div className="flex items-center justify-between mb-5">
          <h3 className="text-xl font-bold">Nova meta</h3>
          <button onClick={onClose} className="p-1.5 hover:bg-secondary rounded-lg">
            <X className="w-5 h-5" />
          </button>
        </div>

        <div className="space-y-4">
          <div>
            <label className="text-xs text-muted-foreground font-semibold block mb-1.5">NOME</label>
            <Input value={item} onChange={(e) => setItem(e.target.value)}
                   placeholder="Ex: Viagem para Tokyo"
                   className="h-11 rounded-xl bg-secondary/50 border-border/40" />
          </div>
          <div className="grid grid-cols-2 gap-3">
            <div>
              <label className="text-xs text-muted-foreground font-semibold block mb-1.5">VALOR ALVO</label>
              <CurrencyInput value={total} onChange={setTotal} placeholder="R$ 0,00"
                             className="h-11 rounded-xl bg-secondary/50 border-border/40" />
            </div>
            <div>
              <label className="text-xs text-muted-foreground font-semibold block mb-1.5">DATA FINAL</label>
              <Input type="date" value={deadline} min={todayISO()}
                     onChange={(e) => setDeadline(e.target.value)}
                     className="h-11 rounded-xl bg-secondary/50 border-border/40" />
            </div>
          </div>
          <div>
            <label className="text-xs text-muted-foreground font-semibold block mb-1.5">CATEGORIA</label>
            <div className="flex flex-wrap gap-2">
              {CATEGORIES.map((c) => (
                <button key={c} onClick={() => setCategory(c)}
                        className={`rounded-full px-3 py-1.5 text-xs font-semibold border transition-all ${
                          category === c
                            ? "bg-primary/20 border-primary/50 text-primary"
                            : "bg-secondary/40 border-border/30 text-muted-foreground"
                        }`}>
                  {CATEGORY_ICONS[c]} {c}
                </button>
              ))}
            </div>
          </div>
        </div>

        <Button disabled={!valid}
                className="mt-6 w-full h-11 rounded-xl bg-gradient-primary text-primary-foreground font-semibold disabled:opacity-50"
                onClick={() => valid && onAdd(item.trim(), total, deadline, category)}>
          Criar meta
        </Button>
      </div>
    </div>
  );
};

// ─── EXCHANGE RATES HOOK ─────────────────────────────────────────────────────

function useExchangeRates() {
  const [rates, setRates] = useState<ExchangeRate[]>([]);
  const [loading, setLoading] = useState(true);
  const [lastUpdated, setLastUpdated] = useState<Date | null>(null);
  const [error, setError] = useState<string | null>(null);

  const fetchRates = async () => {
    setLoading(true);
    try {
      setError(null);
      const res = await fetch(
        "https://economia.awesomeapi.com.br/json/last/USD-BRL,EUR-BRL,GBP-BRL,JPY-BRL,BTC-BRL",
        { cache: "no-store" },
      );
      if (!res.ok) throw new Error(`HTTP ${res.status}`);
      const data = await res.json();
      const mapped: ExchangeRate[] = Object.entries(data).map(([key, v]) => {
        const val = v as { bid: string; pctChange: string; create_date: string };
        return {
          pair: key.replace("BRL", ""),
          rate: parseFloat(val.bid),
          change: parseFloat(val.pctChange),
          createDate: val.create_date,
        };
      });
      const order = ["USD", "EUR", "GBP", "JPY", "BTC"];
      mapped.sort((a, b) => order.indexOf(a.pair) - order.indexOf(b.pair));
      setRates(mapped);
      setLastUpdated(new Date());
    } catch (err) {
      setError(err instanceof Error ? err.message : String(err));
    } finally {
      setLoading(false);
    }
  };

  useEffect(() => {
    fetchRates();
    const id = setInterval(fetchRates, 30 * 1000);
    return () => clearInterval(id);
  }, []);

  return { rates, loading, lastUpdated, error, refetch: fetchRates };
}

// ─── NEWS HOOK ───────────────────────────────────────────────────────────────

function useNews() {
  const [items, setItems] = useState<NewsItem[]>([]);
  const [loading, setLoading] = useState(true);
  const [fetchedAt, setFetchedAt] = useState<number | null>(null);
  const [error, setError] = useState<string | null>(null);

  const load = async (force = false) => {
    setLoading(true);
    const r = await fetchNews(force);
    setItems(r.items);
    setFetchedAt(r.fetchedAt);
    setError(r.error ?? null);
    setLoading(false);
  };

  useEffect(() => { load(false); }, []);

  return { items, loading, fetchedAt, error, refetch: () => load(true) };
}

// ─── MAIN ────────────────────────────────────────────────────────────────────

const TAB_TO_MOOD: Record<Exclude<Tab, "home">, MoodPresetId> = {
  help: "foco",
  investments: "estrategia",
  news: "acao",
  evolution: "estrategia",
};

const Index = () => {
  const store = useGoalStore();
  const { setTheme } = useTheme();
  const [activeTab, setActiveTab] = useState<Tab>("home");
  const [showNewGoal, setShowNewGoal] = useState(false);
  const [showSettings, setShowSettings] = useState(false);
  const [userName, setUserName] = useState(() => localStorage.getItem("user_name") || "Você");
  const [tempName, setTempName] = useState(userName);
  const [confettiKey, setConfettiKey] = useState<number | null>(null);
  const [suggestion, setSuggestion] = useState<MoodPresetId | null>(null);
  const tabTimerRef = useRef<number | null>(null);

  const { rates, lastUpdated: ratesUpdated, loading: loadingRates, refetch: refetchRates } = useExchangeRates();
  const { items: news, loading: loadingNews, fetchedAt: newsAt, error: newsError, refetch: refetchNews } = useNews();

  // Notification badge: unread relevant news + active rate alerts
  const relevantNews = useMemo(() => getRelevantNews(news, store.goals), [news, store.goals]);
  const rateAlerts = useMemo(() => getRateAlerts(rates, store.goals), [rates, store.goals]);
  const [seen, setSeen] = useState<Set<string>>(() => getSeenIds());
  const unreadNews = relevantNews.filter((n) => !seen.has(n.id));
  const unreadCount = unreadNews.length + rateAlerts.length;

  // Marca como visto ao abrir aba de notícias
  useEffect(() => {
    if (activeTab === "news" && unreadNews.length > 0) {
      markSeen(unreadNews.map((n) => n.id));
      setTimeout(() => setSeen(getSeenIds()), 100);
    }
    // eslint-disable-next-line react-hooks/exhaustive-deps
  }, [activeTab, news.length]);

  // Sugestão inteligente: se o usuário ficar 25s+ na mesma aba, sugere o mood ideal.
  useEffect(() => {
    if (tabTimerRef.current) window.clearTimeout(tabTimerRef.current);
    setSuggestion(null);
    if (activeTab === "home") return;
    const mood = TAB_TO_MOOD[activeTab];
    const ctx = `${activeTab}:${mood}`;
    if (getActivePresetId() === mood) return;
    if (wasSuggestionDismissed(ctx)) return;
    tabTimerRef.current = window.setTimeout(() => setSuggestion(mood), 25000);
    return () => {
      if (tabTimerRef.current) window.clearTimeout(tabTimerRef.current);
    };
  }, [activeTab]);

  const acceptSuggestion = () => {
    if (!suggestion) return;
    const m = MOOD_PRESETS.find((p) => p.id === suggestion);
    if (!m) return;
    setTheme(m.theme);
    setActivePresetId(suggestion);
    toast({ title: `${m.emoji} ${m.name} ativado` });
    setSuggestion(null);
  };

  const dismissCurrentSuggestion = () => {
    if (!suggestion || activeTab === "home") { setSuggestion(null); return; }
    dismissSuggestion(`${activeTab}:${suggestion}`);
    setSuggestion(null);
  };

  // Depósito com detecção de meta concluída → confete
  const handleDeposit = (goalId: string, amount: number) => {
    const goal = store.goals.find((g) => g.id === goalId);
    if (!goal) return;
    const wasDone = goalProgress(goal) >= 100;
    const { newBadges } = store.addDeposit(goalId, amount);
    toast({ title: `+${formatBRL(amount)} 💰` });
    // Verifica se agora completou
    const newProgress = ((goalSaved(goal) + amount) / goal.total) * 100;
    if (!wasDone && newProgress >= 100) {
      setConfettiKey(Date.now());
      setTimeout(() => toast({ title: "🏆 Meta conquistada!", description: goal.item }), 300);
    }
    newBadges.forEach((b, i) =>
      setTimeout(() => toast({ title: `${b.icon} ${b.label}`, description: b.desc }), 800 + i * 400),
    );
  };

  const handleAddGoal = (item: string, total: number, deadline: string, category: Category) => {
    if (store.goals.length >= MAX_GOALS) {
      toast({ title: "Limite atingido", description: `Máximo de ${MAX_GOALS} metas ativas.` });
      setShowNewGoal(false);
      return;
    }
    const { newBadges } = store.addGoal({ item, total, deadline, category });
    toast({ title: `Meta criada! ${CATEGORY_ICONS[category]} ${item}` });
    newBadges.forEach((b, i) =>
      setTimeout(() => toast({ title: `${b.icon} ${b.label}`, description: b.desc }), 800 + i * 400),
    );
    setShowNewGoal(false);
  };

  const saveSettings = () => {
    setUserName(tempName);
    localStorage.setItem("user_name", tempName);
    setShowSettings(false);
  };

  const totalSaved = store.dashboard.totalSaved;
  const totalTarget = store.dashboard.totalTarget;

  return (
    <div className="min-h-screen text-foreground">
      {confettiKey && <Confetti key={confettiKey} onDone={() => setConfettiKey(null)} />}

      {/* HEADER */}
      <header className="sticky top-0 z-40 border-b border-primary/15 backdrop-blur-md"
              style={{ background: "linear-gradient(180deg, hsl(152 76% 44% / 0.12), transparent)" }}>
        <div className="max-w-2xl mx-auto px-5 py-5">
          <div className="flex items-start justify-between mb-3">
            <div className="flex items-center gap-3">
              <div className="w-12 h-12 rounded-2xl bg-gradient-primary flex items-center justify-center shadow-emerald">
                <Target className="w-6 h-6 text-primary-foreground" strokeWidth={2.5} />
              </div>
              <div>
                <h1 className="text-xl font-bold tracking-tight">SENTINELA</h1>
                <p className="text-xs text-muted-foreground">Olá, {userName}</p>
              </div>
            </div>
            <button onClick={() => { setTempName(userName); setShowSettings(true); }}
                    className="p-2 hover:bg-secondary rounded-xl transition-colors">
              <Settings className="w-5 h-5" />
            </button>
          </div>

          {store.streak > 0 && (
            <div className="inline-flex items-center gap-2 px-3 py-1.5 rounded-full text-xs font-semibold bg-orange-500/15 border border-orange-500/25 text-orange-300">
              <Flame className="w-4 h-4" />
              {store.streak} dias seguidos
            </div>
          )}
        </div>
      </header>

      {/* MAIN */}
      <main className="max-w-2xl mx-auto px-5 py-6 pb-28">
        {suggestion && (() => {
          const m = MOOD_PRESETS.find((p) => p.id === suggestion);
          if (!m) return null;
          return (
            <div className="mb-4 bg-gradient-card border border-primary/30 rounded-2xl p-4 shadow-card animate-fade-in">
              <div className="flex items-start gap-3">
                <div className="text-2xl">{m.emoji}</div>
                <div className="flex-1 min-w-0">
                  <p className="text-sm font-semibold leading-snug">
                    Você rende mais com este estilo. Deseja ativar?
                  </p>
                  <p className="text-[11px] text-muted-foreground mt-0.5">
           
