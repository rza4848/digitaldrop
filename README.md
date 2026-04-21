# 🚀 DigitalDrop — Boutique de Produits Digitaux

One-page store hébergée sur **GitHub Pages**, avec **Supabase** comme base de données
et intégration **Stripe / PayPal / Wave / Orange Money**.

---

## 📁 Fichiers

```
digitaldrop/
├── index.html           ← Site complet (1 fichier)
├── supabase-schema.sql  ← Schéma base de données
└── README.md
```

---

## ⚙️ Installation en 4 étapes

### 1. GitHub Pages (hébergement gratuit)

```bash
# Créer un repo GitHub, puis :
git init
git add .
git commit -m "Initial commit"
git branch -M main
git remote add origin https://github.com/VOTRE_USERNAME/digitaldrop.git
git push -u origin main
```

Dans GitHub → **Settings → Pages → Source : main branch** → votre site est live !

---

### 2. Supabase (base de données)

1. Créer un compte sur [supabase.com](https://supabase.com) (gratuit)
2. Nouveau projet → noter l'**URL** et la **clef anon**
3. Aller dans **SQL Editor** → coller le contenu de `supabase-schema.sql` → Run
4. Dans `index.html`, remplacer :

```javascript
const SUPABASE_URL  = 'https://VOTRE_URL.supabase.co';   // ← votre URL
const SUPABASE_ANON = 'VOTRE_CLEF_ANON';                 // ← votre clef
```

---

### 3. Stripe (paiement par carte)

1. Compte sur [stripe.com](https://stripe.com)
2. Dashboard → **Developers → API Keys** → copier la clef publique `pk_test_...`
3. Dans `index.html` :

```javascript
const STRIPE_PK = 'pk_test_VOTRE_CLEF_STRIPE';
```

4. Créer une **Edge Function Supabase** pour traiter le paiement côté serveur :

```typescript
// supabase/functions/create-payment/index.ts
import Stripe from 'https://esm.sh/stripe@14'
const stripe = new Stripe(Deno.env.get('STRIPE_SECRET_KEY'))

Deno.serve(async (req) => {
  const { amount, currency, product_name } = await req.json()
  const session = await stripe.checkout.sessions.create({
    payment_method_types: ['card'],
    line_items: [{ price_data: {
      currency: 'xof', unit_amount: amount * 100,
      product_data: { name: product_name }
    }, quantity: 1 }],
    mode: 'payment',
    success_url: `${req.headers.get('origin')}?success=1`,
    cancel_url:  `${req.headers.get('origin')}?canceled=1`,
  })
  return new Response(JSON.stringify({ url: session.url }))
})
```

---

### 4. Autres méthodes de paiement

#### 💛 Wave / Orange Money / Moov
- **CinetPay** (supporte toute l'Afrique de l'Ouest) : [cinetpay.com](https://cinetpay.com)
- **Fedapay** (Bénin, Niger, Togo, Côte d'Ivoire) : [fedapay.com](https://fedapay.com)
- Remplacer la fonction `processPayment()` avec leur SDK

#### 🅿️ PayPal
```html
<script src="https://www.paypal.com/sdk/js?client-id=VOTRE_CLIENT_ID&currency=USD"></script>
```

---

## 📧 Envoi automatique du lien de téléchargement

Utiliser **Supabase Database Webhooks** + **Resend** ou **Mailgun** :

```sql
-- Trigger Supabase : envoyer email quand order.status = 'completed'
create or replace function notify_order_complete()
returns trigger as $$
begin
  perform net.http_post(
    url := 'https://api.resend.com/emails',
    headers := '{"Authorization": "Bearer VOTRE_CLE_RESEND"}'::jsonb,
    body := json_build_object(
      'from', 'noreply@digitaldrop.com',
      'to', NEW.buyer_email,
      'subject', 'Votre téléchargement est prêt !',
      'html', '<p>Merci ' || NEW.buyer_name || ' ! <a href="...">Télécharger</a></p>'
    )::text
  );
  return NEW;
end;
$$ language plpgsql;
```

---

## 🛡️ Sécurité

- Les clefs **secrètes** (Stripe SK, etc.) ne doivent **jamais** être dans `index.html`
- Utilisez les **Edge Functions Supabase** pour tout traitement côté serveur
- Activez le **RLS (Row Level Security)** dans Supabase (déjà dans le schéma SQL)

---

## 🌍 Stack technique

| Couche | Technologie | Coût |
|--------|-------------|------|
| Hébergement | GitHub Pages | Gratuit |
| Base de données | Supabase | Gratuit jusqu'à 500 MB |
| Paiement carte | Stripe | 2.9% + 0.30$ par transaction |
| Paiement mobile | CinetPay / FedaPay | Variable |
| E-mails | Resend | 3 000/mois gratuit |

---

*Créé avec ❤️ pour les entrepreneurs digitaux africains*
