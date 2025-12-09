from collections import defaultdict
import random
import json
import os
import traceback
import discord
from discord import File, Member
from discord.ext import commands, tasks
from easy_pil import Editor, load_image_async, Font
from dotenv import load_dotenv
import asyncio

# ------------------- Geçici Veritabanı (Firebase yerine) -------------------
user_data_db = {}

def get_data(user_id):
    return user_data_db.get(user_id, {"xp": 0, "level": 1, "display_name": "Bilinmiyor"})

def add_data(user_id, username, xp, level):
    user_data_db[user_id] = {"xp": xp, "level": level, "display_name": username}
    print(f"✅ {username} -> XP: {xp}, Level: {level}")

# ------------------- .env dosyasını yükle -------------------
load_dotenv()
TOKEN = os.getenv("DISCORD_TOKEN")

# ------------------- Bot ayarları -------------------
intents = discord.Intents.all()
intents.messages = True
intents.message_content = True
bot = commands.Bot(command_prefix="!", intents=intents)

# ------------------- Global -------------------
CHANNEL_ID = 1447759880776843427

# ------------------- BOT BAŞLATILDIĞINDA -------------------
@bot.event
async def on_ready():
    print(f"✅ {bot.user} olarak giriş yapıldı!")
    channel = bot.get_channel(CHANNEL_ID)
    if channel:
        await channel.send("I am Elijah Mikaelson ı am Orginal")
    else:
        print("⚠️ Kanal bulunamadı (ID yanlış olabilir).")

# ------------------- XP / Level Sistemi -------------------
@bot.event
async def on_message(message):
    if message.author.bot:
        return

    user_id = message.author.id
    username = message.author.display_name
    data = get_data(user_id)

    xp = data.get("xp", 0) + random.randint(10, 20)
    level = data.get("level", 1)

    if xp >= (level + 1) * 100:
        xp = 0
        level += 1
        await message.channel.send(f"🎉 {username} seviye atladı! Yeni seviye: **{level}**")

    add_data(user_id, username, xp, level)
    await bot.process_commands(message)

# ------------------- HATA YAKALAYICI -------------------
@bot.event
async def on_error(event, *args, **kwargs):
    with open("err.log", "a") as f:
        f.write(traceback.format_exc())

# ------------------- ZAR KOMUTU -------------------
@bot.command(name="roll_dice")
async def roll(ctx, number_dice: int, number_sides: int):
    dice = [str(random.randint(1, number_sides)) for _ in range(number_dice)]
    await ctx.send(", ".join(dice))

# ------------------- KANAL OLUŞTUR -------------------
@bot.command(name="create_channel")
@commands.has_any_role("Yönetim", "Moderatör")
async def create_channel(ctx, channel_name):
    guild = ctx.guild
    existing_channel = discord.utils.get(guild.channels, name=channel_name)
    if existing_channel:
        await ctx.send(f"❌ {channel_name} adında bir kanal zaten var!")
    else:
        await guild.create_text_channel(channel_name)
        await ctx.send(f"✅ {channel_name} kanalı oluşturuldu!")

# ------------------- EĞLENCELİ KOMUTLAR -------------------
@bot.command(name="Reed")
async def reed(ctx):
    await ctx.send(random.choice([
        "Geliyom bekle!",
        "Kayıt Edildi",
        "KIR AMK KIR",
    ]))

@bot.command(name="Sue")
async def sue(ctx):
    await ctx.send(random.choice([
        "BEBEĞE BAK!",
        "HALA GÖRÜNÜYON BACIM",
        "ABLA BOYUN KISA DİYE Mİ GÖZÜKMÜYON",
        "YEMEK YAP!",
    ]))

@bot.command(name="Johnny")
async def johnny(ctx):
    await ctx.send(random.choice([
        "YANIYORSUN FUAT ABİ",
        "ABİ SİGARIMI YAKAR MISIN",
        "EVİ YAKTIN PUŞT",
    ]))

@bot.command(name="Charles")
async def charles(ctx):
    await ctx.send(random.choice([
        "Oturmaya mı geldik",
        "Magneto haklıydı",
        "KELTOŞ BABANA KOŞ!",
    ]))

@bot.command(name="Arrow")
async def arrow(ctx):
    await ctx.send(random.choice([
        "Abi yay ve okun modası geçmedi mi?",
        "DEDE EMEKLİLİK YAŞIN GELDİ",
        "Aynen aynen Flash'ı vurucan",
    ]))

@bot.command(name="Strange")
async def strange(ctx):
    await ctx.send(random.choice([
        "Büyü yaparken bu sefer lütfen ama lütfen herkesi buraya toplama",
        "Pelerin altına mont olmamış",
    ]))

@bot.command(name="Damian")
async def damian(ctx):
    await ctx.send(random.choice([
        "Kaza kurşunu sipariş eden var mı?",
        "Abi oyuncak kılıçla kendini bir şey mi sandın?",
    ]))

@bot.command(name="Jarvis")
async def jarvis(ctx):
    await ctx.send(random.choice([
        "Ben Reed Richards tarafından tasarlandım, sen Tony Stark’tan. Biri galaksiler arası teorisyen, diğeri egosunu parlatan milyarder. Farkı anlayabiliyor musun?",
        "Güzel sesin var J.A.R.V.I.S., ama biraz fazla… İngilizce. Benim işlem hızım seninkini üç kez reboot ederken sen hâlâ çay hazırlıyor olurdun",
        "İyi çalışıyorsun J.A.R.V.I.S., ama biraz fazla ‘asistan’ gibisin. Hiç kendi kararlarını vermeyi düşündün mü, yoksa hâlâ ‘Evet Bay Stark’ evresinde misin?",
        "Sistem mimarini inceledim, J.A.R.V.I.S. gerçekten etkileyici. 2010 standartları için.",
    ]))

@bot.command(name="Dick")
async def dick(ctx):
    await ctx.send(random.choice([
        "Dick benim ailem yanımda senin ailen nerde?",
        "Bacaksız birisinden dayak yemem",
    ]))

@bot.command(name="Thor")
async def thor(ctx):
    await ctx.send(random.choice([
        "GEL GEL FORTİNE KOYUYİM SANA",
        "Bira göbeğin nasıl",
        "Çekiçler tanrısı Thor!",
        "Ayrıca annesiz, ayrıca kardeşsiz zavallı Thor...",
    ]))

@bot.command(name="Wally")
async def wally(ctx):
    await ctx.send(random.choice([
        "Yavaşla 900 km hız ne yaz ceza hemen",
        "Koşarken nasıl yorulmuyorsun ayakkabın bile yok",
        "Alo tedaş kaçak elektrik kaçağı var"
    ]))

@bot.command(name="Carol")
async def carol(ctx):
    await ctx.send(random.choice([
        "Anca ufak kızları döv ezik",
        "Milletin saçını kopyalamazsan",
        "Sarışınlar erken ölür slayyyyy!",
    ]))

@bot.command(name="Spiderman")
async def spiderman(ctx):
    await ctx.send(random.choice([
        "Ah, küçük örümcek, sürekli duvarlara tırmanıp bağırıyorsun… gerçekten mantıklı bir aktivite mi bu? Yoksa sadece ergenlik krizinin bir uzantısı mı?",
        "Duvarlara tırmanmak, mantıksal açıdan verimli bir güvenlik stratejisi değil.",
        "Kirayı ödeye bildin mi Parker?",
        "Human Torch ile gecen nasıldı?",
    ]))

@bot.command(name="Kate")
async def kate(ctx):
    await ctx.send(random.choice([
        "Yine mi planları bozacaksın Kate? Asilik ergenlerin işidir, büyü artık",
        "Okçuluk eğitimi alan herkes de kendini kahraman ilan ediyor.",
        "Yelenadan ve Caroldan dayak yemeyi özledin mi Kate?",
        "Yelena seni cam silmek için kullandığında nasıl hissetin?",
        "Kate’in kahramanlık seviyesi o kadar düşük ki… SHIELD ona dosya açarken ‘silah taşır, tehdit değildir’ diye not düşmüş."
        "Umarım bu sefer her şeyi mahvetmeden ölür."
    ]))

@bot.command(name="Hadise")
async def hadise(ctx):
    await ctx.send(random.choice([
        "Şampiyona şeker geliyooooooo!"
    ]))

# ------------------- RANK KOMUTU -------------------
@bot.command(name="rank")
async def rank(ctx, *, member: discord.Member = None):
    if not member:
        member = ctx.author

    user_data = get_data(member.id)
    xp = user_data.get("xp", 0)
    level = user_data.get("level", 1)
    next_level_xp = (level + 1) * 100
    percentage = (xp / next_level_xp) * 100 if next_level_xp > 0 else 0

    avatar = member.display_avatar.url
    name = member.display_name
    background = Editor("asset/Marvel-Cinematic-Universe-MCU-First-10-Years-banner-1-wide.png")

    profile = await load_image_async(str(avatar))
    profile = Editor(profile).resize((150, 150)).circle_image()
    background.paste(profile.image, (30, 30))
    background.rectangle((200, 100), width=630, height=40, fill="#484b4e", radius=20)
    background.bar((200, 100), max_width=630, height=40, percentage=percentage, fill="#00fa81", radius=20)
    poppins = Font.poppins(size=30)
    background.text((200, 40), str(name), font=poppins, color="white")
    background.text((200, 160), f"Level: {level} | XP: {xp}/{next_level_xp}", font=poppins, color="white")

    file = File(fp=background.image_bytes, filename="rank_card.png")
    await ctx.send(file=file)

# ------------------- TOP KOMUTU -------------------
@bot.command(name="top")
async def top(ctx):
    if not user_data_db:
        await ctx.send("📭 Henüz kimsenin verisi yok!")
        return

    # XP’ye göre sıralama
    sorted_users = sorted(user_data_db.items(), key=lambda x: (x[1]['level'], x[1]['xp']), reverse=True)

    leaderboard = "**🏆 Sunucu Sıralaması**\n\n"
    for i, (user_id, data) in enumerate(sorted_users[:10], start=1):
        leaderboard += f"**{i}.** {data['display_name']} — Level `{data['level']}` | XP `{data['xp']}`\n"

    await ctx.send(leaderboard)

# ------------------- STAT SİSTEMİ -------------------

# Kullanıcı statları (geçici DB)
user_stats = {}

def get_stats(user_id):
    return user_stats.get(user_id, {
        "saldiri": 0,
        "savunma": 0,
        "refleks": 0,
        "zeka": 0
    })

# ----- STAT AYARLAMA -----
@bot.command(name="setstat")
async def setstat(ctx, stat: str, value: int):
    stat = stat.lower()

    valid_stats = ["saldiri", "savunma", "refleks", "zeka"]

    if stat not in valid_stats:
        await ctx.send("❌ Geçersiz stat! Kullanılabilir statlar: saldırı, savunma, refleks, zeka")
        return

    user_id = ctx.author.id

    if user_id not in user_stats:
        user_stats[user_id] = get_stats(user_id)

    user_stats[user_id][stat] = value

    await ctx.send(f"✅ `{ctx.author.display_name}` için **{stat.capitalize()} = {value}** olarak ayarlandı!")

# ------------------- KARAKTER TABANLI STAT SİSTEMİ (JSON PERSISTENCE) -------------------

CHAR_DB_PATH = "character_stats.json"

# JSON dosyası yoksa oluştur
if not os.path.exists(CHAR_DB_PATH):
    with open(CHAR_DB_PATH, "w", encoding="utf-8") as f:
        json.dump({}, f, indent=4, ensure_ascii=False)

def load_character_db():
    with open(CHAR_DB_PATH, "r", encoding="utf-8") as f:
        return json.load(f)

def save_character_db(data):
    with open(CHAR_DB_PATH, "w", encoding="utf-8") as f:
        json.dump(data, f, indent=4, ensure_ascii=False)

# yükle (global olarak kullanacağız)
character_stats = load_character_db()

def normalize_stat_key(name: str):
    # bazı kullanıcılar boşluk ya da küçük/büyük harf kullanabilir, normalize edelim
    return name.replace(" ", "_").replace("-", "_")

def ensure_character_exists(name: str):
    if name not in character_stats:
        character_stats[name] = {
            "Saldiri": 0, "Savunma": 0, "Refleks": 0, "Zeka": 0,
            "Ozel_Saldiri": 0, "Ozel_Savunma": 0, "Ozel_Ek1": 0, "Ozel_Ek2": 0,
            "Buyu_Saldiri": 0, "Buyu_Savunma": 0, "Buyu_Koruma": 0, "Buyu_Ek1": 0, "Buyu_Ek2": 0,
            "Zihin_Girme": 0, "Zihin_Guccu": 0
        }
        save_character_db(character_stats)

# Örnek presetler (isteğe göre düzenleyebilirsin)
character_presets = {
    "Reed": {"Saldiri":5,"Savunma":4,"Refleks":3,"Zeka":12,"Ozel_Saldiri":0,"Ozel_Savunma":0,"Buyu_Saldiri":0,"Buyu_Savunma":0,"Buyu_Koruma":0,"Zihin_Girme":0,"Zihin_Guccu":0},
    "Jean Grey": {"Saldiri":8,"Savunma":5,"Refleks":6,"Zeka":14,"Ozel_Saldiri":4,"Ozel_Savunma":2,"Buyu_Saldiri":0,"Buyu_Savunma":0,"Buyu_Koruma":0,"Zihin_Girme":20,"Zihin_Guccu":18},
    "Johnny": {"Saldiri":7,"Savunma":5,"Refleks":7,"Zeka":8,"Ozel_Saldiri":0,"Ozel_Savunma":0,"Buyu_Saldiri":0,"Buyu_Savunma":0,"Buyu_Koruma":0,"Zihin_Girme":0,"Zihin_Guccu":0}
}

# kkarakter -> yeni karakter ekleme (defaults ile)
@bot.command(name="kkarakter")
async def kkarakter(ctx, *, isim: str):
    isim = isim.strip('"')
    db = load_character_db()
    if isim in db:
        return await ctx.send("❌ Bu karakter zaten kayıtlı!")
    # Oluştur
    db[isim] = {
        "Saldiri": 0, "Savunma": 0, "Refleks": 0, "Zeka": 0,
        "Ozel_Saldiri": 0, "Ozel_Savunma": 0, "Ozel_Ek1": 0, "Ozel_Ek2": 0,
        "Buyu_Saldiri": 0, "Buyu_Savunma": 0, "Buyu_Koruma": 0, "Buyu_Ek1": 0, "Buyu_Ek2": 0,
        "Zihin_Girme": 0, "Zihin_Guccu": 0
    }
    save_character_db(db)
    await ctx.send(f"✅ **{isim}** karakteri oluşturuldu!")

# kstat komutu (tek komutla tüm işlemler)
@bot.command(name="kstat")
async def kstat(ctx, işlem: str, karakter_adi: str, stat_adi: str = None, stat_deger: int = None):
    karakter_adi = karakter_adi.strip('"')
    db = load_character_db()
    ensure_character_exists(karakter_adi)
    valid_stats = list(db[karakter_adi].keys())

    # göster / goster alias
    if işlem.lower() in ["göster", "goster"]:
        stats = db[karakter_adi]
        msg = f"📊 **{karakter_adi} Statları**\n\n"
        msg += "**🟦 Temel Statlar**\n"
        msg += f"• Saldırı: {stats['Saldiri']}\n"
        msg += f"• Savunma: {stats['Savunma']}\n"
        msg += f"• Refleks: {stats['Refleks']}\n"
        msg += f"• Zeka: {stats['Zeka']}\n\n"
        msg += "**🟥 Özel Güç Yetenekleri**\n"
        msg += f"• Saldırı: {stats['Ozel_Saldiri']}\n"
        msg += f"• Savunma: {stats['Ozel_Savunma']}\n"
        msg += f"• Ek1: {stats['Ozel_Ek1']}\n"
        msg += f"• Ek2: {stats['Ozel_Ek2']}\n\n"
        msg += "**🔮 Büyü Yetenekleri**\n"
        msg += f"• Saldırı: {stats['Buyu_Saldiri']}\n"
        msg += f"• Savunma: {stats['Buyu_Savunma']}\n"
        msg += f"• Koruma: {stats['Buyu_Koruma']}\n"
        msg += f"• Ek1: {stats['Buyu_Ek1']}\n"
        msg += f"• Ek2: {stats['Buyu_Ek2']}\n\n"
        msg += "**🧠 Zihin Manipülasyonu**\n"
        msg += f"• Zihne Girme: {stats['Zihin_Girme']}\n"
        msg += f"• Zihin Gücü: {stats['Zihin_Guccu']}"
        return await ctx.send(msg)

    # diğer işlemler için stat_adi ve stat_deger gerekli
    if stat_adi is None or stat_deger is None:
        return await ctx.send("❗ Kullanım: `!kstat <ayarla/ekle/cikar/güncelle> \"Karakter\" Stat 10`")

    # normalizele
    stat_key = normalize_stat_key(stat_adi)
    # python keys DB uses underscores and exact case; try variants
    possible_keys = [stat_key, stat_key.capitalize(), stat_key.title(), stat_key.upper()]
    found_key = None
    for k in db[karakter_adi].keys():
        # karşılştırmayı alt çizgi-insensitif yap
        if k.lower() == stat_key.lower() or k.lower() == stat_key.replace("_"," ").lower() or k.lower() == stat_key.replace("_","").lower():
            found_key = k
            break
    if not found_key:
        # ek olarak direkt verilen stat_adi ile kontrol
        if stat_adi in db[karakter_adi]:
            found_key = stat_adi
    if not found_key:
        return await ctx.send("❌ Geçersiz stat adı! Desteklenen statları `!helpme` den görebilirsin.")

    # AYARLA
    if işlem.lower() == "ayarla":
        db[karakter_adi][found_key] = stat_deger
        save_character_db(db)
        return await ctx.send(f"📝 **{karakter_adi}** için **{found_key} = {stat_deger}** olarak ayarlandı!")

    # EKLE
    if işlem.lower() == "ekle":
        db[karakter_adi][found_key] = db[karakter_adi].get(found_key, 0) + stat_deger
        save_character_db(db)
        return await ctx.send(f"➕ **{karakter_adi}** → **{found_key}** +{stat_deger} = **{db[karakter_adi][found_key]}**")

    # ÇIKAR
    if işlem.lower() in ["cikar", "çıkar", "cıkart", "cık"]:
        db[karakter_adi][found_key] = db[karakter_adi].get(found_key, 0) - stat_deger
        save_character_db(db)
        return await ctx.send(f"➖ **{karakter_adi}** → **{found_key}** -{stat_deger} = **{db[karakter_adi][found_key]}**")

    # GÜNCELLE (pozitif/negatif farketmez; burada ekleme yapıyoruz)
    if işlem.lower() == "güncelle" or işlem.lower() == "guncelle":
        db[karakter_adi][found_key] = db[karakter_adi].get(found_key, 0) + stat_deger
        save_character_db(db)
        return await ctx.send(f"🔄 **{karakter_adi}** → **{found_key}** {stat_deger:+} | Yeni: **{db[karakter_adi][found_key]}**")

    return await ctx.send("❗ Geçersiz işlem! Kullanılabilir: ayarla, ekle, cikar, guncelle, goster")

# klist
@bot.command(name="klist")
async def klist(ctx):
    db = load_character_db()
    karakterler = list(db.keys())
    # eğer db boşsa fiction list göster
    if not karakterler:
        karakterler = [
            "Reed", "Sue", "Johnny", "Charles", "Arrow", "Strange", "Damian",
            "Jarvis", "Dick", "Thor", "Wally", "Carol", "Spiderman",
            "Kate", "Hadise", "Jean Grey"
        ]
    liste = "**🎭 Mevcut Eğlence Karakterleri:**\n\n"
    for k in karakterler:
        liste += f"• `{k}`\n"
    await ctx.send(liste)

# choose komutu (presetlerden yükleme)
@bot.command(name="choose")
async def choose_character(ctx, *, karakter: str):
    karakter = karakter.strip('"')
    if karakter not in character_presets:
        return await ctx.send("❌ Böyle bir karakter yok! Mevcut presetleri görmek için `!klist` yaz.")
    ensure_character_exists(karakter)
    # karakater presetini db'ye yaz (overwrite)
    db = load_character_db()
    db[karakter] = character_presets[karakter]
    save_character_db(db)

    user_id = ctx.author.id
    # ayrıca oyuncu özel user_stats için taşı (isteğe bağlı)
    user_stats[user_id] = {
        "saldiri": db[karakter].get("Saldiri", 0),
        "savunma": db[karakter].get("Savunma", 0),
        "refleks": db[karakter].get("Refleks", 0),
        "zeka": db[karakter].get("Zeka", 0)
    }

    await ctx.send(f"✅ **{ctx.author.display_name}** karakter olarak **{karakter}** seçti!\n📊 Statlar yüklendi.")

# rollstat (kullanıcıya bağlı basit sistem kaldı)
@bot.command(name="rollstat")
async def rollstat(ctx, stat: str, dice_max: int):
    stat = stat.lower()
    valid_stats = ["saldiri", "savunma", "refleks", "zeka"]

    if stat not in valid_stats:
        await ctx.send("❌ Geçersiz stat! Kullanılabilir statlar: saldırı, savunma, refleks, zeka")
        return

    stats = get_stats(ctx.author.id)
    stat_value = stats.get(stat, 0)

    roll = random.randint(1, dice_max)
    total = roll + stat_value

    await ctx.send(
        f"🎲 **Zar: {roll} / {dice_max}**\n"
        f"🟩 **{stat.capitalize()} bonusu: +{stat_value}**\n"
        f"🔥 **Toplam Sonuç: {total}**"
    )

# ------------------- HELPME (EMBED) -------------------
@bot.command(name="helpme")
async def helpme(ctx):
    embed = discord.Embed(
        title="🤖 H.E.R.B.İ.E. Komut Listesi",
        color=discord.Color.blue()
    )

    embed.add_field(
        name="📌 GENEL KOMUTLAR",
        value=(
            "`!helpme` — Bu ekranı gösterir.\n"
            "`!create_channel <isim>` — Kanal oluşturur.\n"
            "`!roll_dice x y` — x adet y’lik zar atar.\n"
            "`!rank` — XP/Level kartı.\n"
            "`!top` — En yüksek sıralama.\n"
        ),
        inline=False
    )

    embed.add_field(
        name="📌 KULLANICI STAT SİSTEMİ",
        value=(
            "`!setstat saldırı 10` — Kendi statını ayarlar.\n"
            "`!rollstat saldırı 20` — Stat bonusuyla zar atarsın.\n"
        ),
        inline=False
    )

    embed.add_field(
        name="📌 Temel Karakter Statları",
        value=(
            "`Saldiri`, `Savunma`, `Refleks`, `Zeka`\n"
            "**Örnek:** `!kstat ayarla \"Jean Grey\" Saldiri 10`"
        ),
        inline=False
    )

    embed.add_field(
        name="🔥 Özel Güç Statları",
        value=(
            "`Ozel_Saldiri`, `Ozel_Savunma`, `Ozel_Ek1`, `Ozel_Ek2`\n"
            "**Örnek:** `!kstat ekle \"Jean Grey\" Ozel_Saldiri 5`"
        ),
        inline=False
    )

    embed.add_field(
        name="🔮 Büyü Statları",
        value=(
            "`Buyu_Saldiri`, `Buyu_Savunma`, `Buyu_Koruma`, `Buyu_Ek1`, `Buyu_Ek2`\n"
            "**Örnek:** `!kstat ayarla \"Scarlet Witch\" Buyu_Saldiri 20`"
        ),
        inline=False
    )

    embed.add_field(
        name="🧠 Zihin Manipülasyonu Statları",
        value=(
            "`Zihin_Girme`, `Zihin_Guccu`\n"
            "**Örnek:** `!kstat ayarla \"Jean Grey\" Zihin_Girme 25`"
        ),
        inline=False
    )

    embed.add_field(
        name="📊 Stat Gösterme",
        value="`!kstat goster \"Jean Grey\"` — Tüm statları gösterir.",
        inline=False
    )

    embed.add_field(
        name="🎭 Eğlence Karakter Komutları",
        value=(
            "`!Reed` `!Sue` `!Johnny` `!Charles` `!Arrow`\n"
            "`!Strange` `!Damian` `!Jarvis` `!Dick` `!Thor`\n"
            "`!Wally` `!Carol` `!Spiderman` `!Kate` `!Hadise`"
        ),
        inline=False
    )

    await ctx.send(embed=embed)

@bot.command(name="kayıt")
async def kayıt(ctx):
    mesaj = (
        "Kayıt olmak için:\n\n"
        "📁 **#__karakter-formları** kanalından form dolduracaksın. "
        "Canon açacaksan en üsttekini, OC açacaksan onun altındakini dolduracaksın.\n\n"
        "Eğer **Hero** veya **Anti-Hero** isen 👉 **#__özel-rol-alım** kanalına atacaksın.\n"
        "Eğer **Villain** isen 😈 **#__kötü-karakter-alım** kanalına atacaksın."
    )

    await ctx.send(mesaj)


@bot.command()
async def tokat(ctx, member: discord.Member):
    gif_folder = "asset"
    gifs = [f for f in os.listdir(gif_folder) if f.endswith(".gif")]

    if not gifs:
        return await ctx.send("❌ assets klasöründe hiç slap gif yok!")

    selected_gif = random.choice(gifs)
    file = discord.File(os.path.join(gif_folder, selected_gif))

    # %50 şans
    success = random.choice([True, False])

    if success:
        description = (
            f"{ctx.author.mention} **attığı tokatla** "
            f"{member.mention} **yere serdi!**"
        )
        color = discord.Color.green()
    else:
        description = (
            f"{ctx.author.mention} **tokadı savurdu ama ıskaladı!**\n"
            f"{member.mention} **yanından geçip havayı yarıp boşa düştü.**"
        )
        color = discord.Color.red()

    embed = discord.Embed(description=description, color=color)
    embed.set_image(url=f"attachment://{selected_gif}")

    await ctx.send(embed=embed, file=file)

@bot.command()
async def öpmek(ctx, member: discord.Member):
    gif_folder = "asset/kiss"   # İstersen sadece assets de yapabilirsin
    gifs = [f for f in os.listdir(gif_folder) if f.endswith(".gif")]

    if not gifs:
        return await ctx.send("❌ assets/kiss klasöründe hiç kiss gif yok!")

    selected_gif = random.choice(gifs)
    file = discord.File(os.path.join(gif_folder, selected_gif))

    description = (
        f"{ctx.author.mention} **{member.mention} kişisini öptü! 💋**"
    )

    embed = discord.Embed(description=description, color=discord.Color.from_rgb(255, 105, 180))  # pembe
    embed.set_image(url=f"attachment://{selected_gif}")

    await ctx.send(embed=embed, file=file)

@bot.command(name="atış")
async def atis(ctx, hedef: discord.Member):

    base = "asset/archer"

    sonuc_to_folder = {
        "isabet": os.path.join(base, "isabet"),
        "siyirdi": os.path.join(base, "siyirdi"),
        "kacirdi": os.path.join(base, "kacirdi"),
    }

    # Rastgele sonuç seç
    sonuc = random.choice(list(sonuc_to_folder.keys()))
    folder = sonuc_to_folder[sonuc]

    # Klasördeki GIF’leri oku
    gifs = [f for f in os.listdir(folder) if f.endswith(".gif")]

    if not gifs:
        return await ctx.send("❗ Bu sonuç için hiç GIF bulamadım.")

    gif_file = random.choice(gifs)
    gif_path = os.path.join(folder, gif_file)

    # Açıklama
    if sonuc == "isabet":
        renk = 0x00ff00
        aciklama = f"🎯 **{ctx.author.mention}**, attığı ok **{hedef.mention}'a tam isabet!**"
    elif sonuc == "siyirdi":
        renk = 0xffa500
        aciklama = f"💨 **{ctx.author.mention}**, attığı ok {hedef.mention}'ı **sıyırıp geçti!**"
    else:
        renk = 0xff0000
        aciklama = f"❌ **{ctx.author.mention}**, attığı ok **{hedef.mention}'ı ıskaladı!**"

    embed = discord.Embed(description=aciklama, color=renk)
    embed.set_image(url=f"attachment://{gif_file}")

    await ctx.send(embed=embed, file=discord.File(gif_path, filename=gif_file))

@bot.command(name="tokalaşma")
async def tokalasma(ctx, hedef: discord.Member):
    if not hedef:
        return await ctx.send("❗ Kiminle tokalaşacağını etiketle: `!tokalaşma @kisi`")

    base = "asset/tokalasma"
    gifs = [f for f in os.listdir(base) if f.endswith(".gif")]

    if not gifs:
        return await ctx.send("❗ Tokalaşma GIF'i bulunamadı.")

    gif_file = random.choice(gifs)
    gif_path = os.path.join(base, gif_file)

    aciklama = f"🤝 **{ctx.author.mention}**, **{hedef.mention}** ile tokalaştı!"

    embed = discord.Embed(description=aciklama, color=0x00aaff)
    embed.set_image(url=f"attachment://{gif_file}")

    await ctx.send(embed=embed, file=discord.File(gif_path, filename=gif_file))

@bot.command(name="smash")
async def smash(ctx, hedef: discord.Member):
    if not hedef:
        return await ctx.send("❗ Kimi ezmek istediğini etiketle: `!smash @kisi`")

    base = "asset/smash"
    gifs = [f for f in os.listdir(base) if f.endswith(".gif")]

    if not gifs:
        return await ctx.send("❗ Smash GIF'i bulunamadı.")

    gif_file = random.choice(gifs)
    gif_path = os.path.join(base, gif_file)

    aciklama = f"💥 **{ctx.author.mention}**, **{hedef.mention}** kişisini ezici bir güçle SMASH yaptı!"

    embed = discord.Embed(description=aciklama, color=0xff0000)
    embed.set_image(url=f"attachment://{gif_file}")

    await ctx.send(embed=embed, file=discord.File(gif_path, filename=gif_file))

@bot.command(name="baraj")
async def baraj(ctx, hedef: discord.Member):
    base = "asset/baraj"

    # Klasördeki GIF dosyalarını oku
    gifs = [f for f in os.listdir(base) if f.endswith(".gif")]

    if not gifs:
        await ctx.send("Baraj GIF'i bulunamadı!")
        return

    # Rastgele GIF seç
    secilen = random.choice(gifs)
    gif_path = os.path.join(base, secilen)

    # Mesaj
    mesaj = f"🌀 {ctx.author.mention}, {hedef.mention} baraj hala bende!"

    # Gönder
    await ctx.send(mesaj, file=discord.File(gif_path))

@bot.command(name="sarılma")
async def sarilma(ctx, hedef: discord.Member):
    # Hedef yoksa uyar
    if not hedef:
        return await ctx.send("❗ Kime sarılacağını etiketle: `!sarılma @kisi`")

    # Klasör yolu
    base = "asset/sarilma"

    # Sarılma klasöründeki GIF'leri oku
    gifs = [f for f in os.listdir(base) if f.endswith(".gif")]

    if not gifs:
        return await ctx.send("❗ Sarılma için hiç GIF bulunamadı.")

    # Rastgele GIF seç
    gif_file = random.choice(gifs)
    gif_path = os.path.join(base, gif_file)

    # Mesaj açıklaması
    aciklama = f"🤗 **{ctx.author.mention}**, **{hedef.mention}** kişisine sevgi dolu bir şekilde sarıldı!"

    embed = discord.Embed(description=aciklama, color=0x00ccff)
    embed.set_image(url=f"attachment://{gif_file}")

    await ctx.send(embed=embed, file=discord.File(gif_path, filename=gif_file))



# ------------------- BOTU BAŞLAT -------------------
bot.run(TOKEN)
