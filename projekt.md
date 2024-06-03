

# GymBro
## Your pocket-friendly Personal Trainer
### En Discord-bot skapad av Calle, Fred & Robin

## ---------------------------------------------

## Importerar moduluerna OS, Discord & Datetime
```python 
import os
from dotenv import load_dotenv
from discord.ext import commands
import discord
from datetime import datetime
```


## Hämtar discord tokenen från .env-fil vi skapat.
```python 
load_dotenv()
TOKEN = os.getenv('DISCORD_TOKEN')
```

## Om den inte finns så kan inte boten köras, då får vi ett felmedellande
```python 
if TOKEN is None:
    raise ValueError("No DISCORD_TOKEN found in environment variables")
```

## Här bestämmer vi vad botten får göra och inte.
```python 
intents = discord.Intents.default()
```

## Detta tillåter botten at läsa meddelande den får skickat till sig.
```python 
intents.message_content = True
```

## Hur vi anropar botten till att börja med, vi måste alltid skriva ! först för att den ska starta.
```python 
bot = commands.Bot(command_prefix='!', intents=intents)
```

## En dictionary som lagrar användaren data.
```python 
user_data = {}
```

## En dicitionary som har olika övningar sparade för våra användare att välja mellan. 
```python 
workouts = {
    "push": ["Bänkpress", "Hantelpress", "Flyes", "Push-ups", "Dips"],
    "pull": ["Rodd", "Marklyft", "Pull-ups", "Chins", "Hantelrodd"],
    "legs": ["Knäböj", "Utfall", "Marklyft", "Benspark", "Bencurl"]
}
```

## Skapar en meny för botten att skriva ut när vi anropar den på rätt sätt.
```python 
def format_menu():
    return ("What are you working out today?\n"
            "1. Push\n"
            "2. Pull\n"
            "3. Legs\n"
            "4. Progress\n"
            "5. Exit")
```

## Funktion som formaterar listan av övningar för en specikt typ av träning.
```python 
def format_exercises(workout_type):
    return "\n".join([f"{i+1}. {exercise}" for i, exercise in enumerate(workouts[workout_type])]) + "\nReturn"
```

## Funktion för att spara en användares tränings
```python 
def save_progress(user_id, workout_type, exercise, weights):
```

## Få nuvarande tid och datum och sparar detta till en variabel för att senare använda vid loggning.
```python 
timestamp = datetime.now().strftime("%Y-%m-%d %H:%M:%S")
```

## Få nuvarande tid och datum och sparar detta till en variabel för att senare använda vid loggning.Formaterar tränigsdata till en ssträng 
```python 
progress_entry = f"{timestamp} - {exercise}: " + ", ".join([f"Set {i+1}: {w['reps']} reps with {w['weight']} kg" for i, w in enumerate(weights)])
```

## Filnamen på den filen vi ska spara till den specifika användaren, blir unik då vi använder user_id i filen.
```python 
progress_file = f"{user_id}_progress.txt" 
```

## Öppnar filen och skriver in datan från tidigare strängar i filen.
```python 
with open(progress_file, "a") as file: 
        file.write(progress_entry + "\n")
```

## Funktion för att läsa användarens träningsfil 
```python 
def load_progress(user_id, exercise):
```

## Skapar filen med användarens user ID.
```python 
progress_file = f"{user_id}_progress.txt" ´
```

## Kontrollerar om filen existerar eller inte.
```python 
if os.path.exists(progress_file):
```

## Öppnar filen som read för att kunna läsa av filens inerhåll.
```python 
with open(progress_file, "r") as file:
```

## Läser igenom filen 
```python 
lines = file.readlines()            
for line in reversed(lines):
```

## Kollar om raden inehåller övningen eller inte
```python 
if f"- {exercise}:" in line: 
```

## Skriver ut raden om den finns
```python 
return line.strip()
```

## Retunerar inget om filen inte hittas.
```python 
return None 
```

## Event som körs när botten har loggat in och är redo
```python 
@bot.event
async def on_ready():
    print(f'Logged in as {bot.user}!')
```

## Kommando som användaren kan använda för att starta botten och lägger till ! i början.
```python 
@bot.command(name='start')
async def start(ctx):
```

## Skicka huvudmenyn till användaren via DM för att skriva det i en privat chatt.
```python 
await ctx.author.send(format_menu())
```

## Spara användarens nuvarande state som "menu" för att den ska veta var användaren befinner sig.
```python 
user_data[ctx.author.id] = {"state": "menu"}  
```

## Event som triggas när ett meddelande skickas
```python 
@bot.event
async def on_message(message):
```

## Kontrollera att meddelandet är en DM och att det inte skickats av en bot
```python 
if message.guild is None and not message.author.bot:
        await handle_dm(message)
```

## Processa kommandon efter att meddelandet har hanterats
```python 
await bot.process_commands(message)
```

## Funktion för att hantera DM-meddelanden
```python 
async def handle_dm(message):
```

## Hämta användarens ID och meddelandets innehåll
```python 
user_id = message.author.id
    content = message.content.lower()
```

## Om användaren inte finns i user_data, be dem starta med !start
```python 
if user_id not in user_data:
    await message.author.send("Please start by typing `!start`.")
        return
```

## Hämta användarens nuvarande state, detta för att veta återigen vart användaren är.
```python 
state = user_data[user_id].get("state")
```

## Baserat på användarens state, hantera deras meddelande då vi nu vet var användaren befinner sig i menyn.
```python 
if state == "menu":
        await handle_menu_selection(message, content)
    elif state == "select_exercise":
        await handle_exercise_selection(message, content)
    elif state == "enter_reps":
        await handle_reps_entry(message, content)
    elif state == "enter_weight":
        await handle_weight_entry(message, content)


async def handle_menu_selection(message, content):
```

## Hämtare användarens ID utefter vem det var som skrev !start
```python 
user_id = message.author.id
```

## Beroende på användarens val från huvudmenyn
```python 
if content == '1':
```

## Skicka Push-träningspassets övningar till användaren
```python 
await message.author.send("Push workout:\n" + format_exercises("push"))
```

## Uppdatera användarens state till "select_exercise" och spara träningspassets typ
```python 
user_data[user_id] = {"state": "select_exercise", "workout_type": "push"}
    elif content == '2':
```

## Skicka Pull-träningspassets övningar till användaren
```python 
await message.author.send("Pull workout:\n" + format_exercises("pull"))
```

## Uppdatera användarens state till "select_exercise" och spara träningspassets typ
```python 
user_data[user_id] = {"state": "select_exercise", "workout_type": "pull"}
    elif content == '3':
```

## Skicka Legs-träningspassets övningar till användaren
```python 
await message.author.send("Legs workout:\n" + format_exercises("legs"))
```

## Uppdatera användarens state till "select_exercise" och spara träningspassets typ
```python 
user_data[user_id] = {"state": "select_exercise", "workout_type": "legs"}
    elif content == '4':
```

## Visa användarens träningsprogress
```python 
await show_progress(message)
    elif content == '5':
```

## Meddela användaren att vila är bra och ta bort deras data från user_data
```python 
await message.author.send("Rest days are good for you!")
        user_data.pop(user_id, None)
    else:
```

## Om användaren anger ett ogiltigt val, skicka ett felmeddelande och visa huvudmenyn igen
```python 
await message.author.send("Invalid selection. " + format_menu())

async def handle_exercise_selection(message, content):
```

## Hämta användarens ID
```python 
user_id = message.author.id
```

## Hämta träningspassets typ från användarens data
```python 
workout_type = user_data[user_id]["workout_type"]
```

## Om användaren väljer att gå tillbaka till huvudmenyn
```python 
if content == 'return':
```

## Skicka huvudmenyn till användaren och uppdatera användarens state
```python 
await message.author.send(format_menu())
        user_data[user_id] = {"state": "menu"}
    else:
        try:
```

## Försök att omvandla användarens inmatning till ett heltal (index för övning)
```python 
exercise_index = int(content) - 1
```

## Hämta den valda övningen
```python 
exercise = workouts[workout_type][exercise_index]
```

## Läs användarens senaste träningsprogress för den valda övningen
```python 
last_weights = load_progress(user_id, exercise)
```

## Skicka användaren deras senaste träningsprogress för den valda övningen
```python 
await message.author.send(f"Last time you did {exercise}, you did:\n{last_weights}" if last_weights else f"No previous data for {exercise}.")
```

## Be användaren ange antal reps för första setet eller spara övningen och gå tillbaka till träningsmenyn
```python 
await message.author.send(f"Enter the number of reps for Set 1 for {exercise} (1-50) or type 'save' to save the exercise and go back to the workout menu.")
```

## Uppdatera användarens state för att reflektera nästa steg i processen
```python 
user_data[user_id] = {"state": "enter_reps", "workout_type": workout_type, "exercise": exercise, "set": 1, "weights": []}
        except (ValueError, IndexError):
```

## Om användaren anger ett ogiltigt val, skicka ett felmeddelande och visa övningarna igen
```python 
await message.author.send("Invalid selection. Please select an exercise by typing its number or type 'return' to go back to the main menu.")

async def handle_reps_entry(message, content):
```

## Hämta användarens ID
```python 
user_id = message.author.id
```

## Om användaren väljer att spara övningen
```python 
if content == 'save':
```

## Anropa funktionen för att spara övningen
```python 
await save_exercise(message)
    else:
        try:
```
## Försök omvandla användarens inmatning till ett heltal (antal reps)
```python 
reps = int(content)
```

## Om antalet reps är giltigt
```python 
if 1 <= reps <= 50:
```

## Be användaren ange vikten för det aktuella setet eller spara övningen och gå tillbaka till träningsmenyn
```python 
await message.author.send(f"Enter the weight for Set {user_data[user_id]['set']} for {user_data[user_id]['exercise']} (1-300kg) or type 'save' to save the exercise and go back to the workout menu.")
```

## Lägg till antalet reps i användarens data och uppdatera state för att reflektera nästa steg
```python 
user_data[user_id]["weights"].append({"reps": reps})
                user_data[user_id]["state"] = "enter_weight"
            else
```

## Om antalet reps är ogiltigt, skicka ett felmeddelande
```python 
await message.author.send("Invalid number of reps. Please enter a number between 1 and 50.")
        except ValueError:
```

## Om inmatningen inte kan omvandlas till ett heltal, skicka ett felmeddelande
```python 
await message.author.send("Invalid input. Please enter a valid number of reps.")

async def handle_weight_entry(message, content):
```

## Hämta användarens ID
```python 
user_id = message.author.id
```

## Om användaren väljer att spara övningen
```python 
if content == 'save':
```

## Anropa funktionen för att spara övningen
```python 
await save_exercise(message)
    else:
        try:
```

## Försök omvandla användarens inmatning till ett flyttal (vikten)
```python 
weight = float(content)
```

## Om vikten är giltig
```python 
if 1 <= weight <= 300:
```

## Uppdatera användarens vikt i det senaste setet och förbered för nästa set
```python 
user_data[user_id]["weights"][-1]["weight"] = weight
                next_set = user_data[user_id]["set"] + 1
```

## Be användaren ange antal reps för nästa set eller spara övningen och gå tillbaka till träningsmenyn
```python 
await message.author.send(f"Enter the number of reps for Set {next_set} for {user_data[user_id]['exercise']} (1-50) or type 'save' to save the exercise and go back to the workout menu.")
```

## Uppdatera state för att reflektera nästa steg
```python 
user_data[user_id]["set"] = next_set
                user_data[user_id]["state"] = "enter_reps"
            else:
```

# Om vikten är ogiltig, skicka ett felmeddelande
```python 
await message.author.send("Invalid weight. Please enter a number between 1 and 300 kg.")
        except ValueError:
```

## Om inmatningen inte kan omvandlas till ett flyttal, skicka ett felmeddelande
```python 
await message.author.send("Invalid input. Please enter a valid weight.")

async def save_exercise(message):
```

## Hämta användarens ID, träningspassets typ, övningen och vikterna
```python 
user_id = message.author.id
    workout_type = user_data[user_id]["workout_type"]
    exercise = user_data[user_id]["exercise"]
    weights = user_data[user_id]["weights"]
```

## Spara övningens progress och meddela användaren
```python 
save_progress(user_id, workout_type, exercise, weights)
    await message.author.send(f"Exercise {exercise} saved. Select another exercise or type 'return' to go back to the workout menu.\n" + format_exercises(workout_type))
```

## Uppdatera state för att reflektera nästa steg
```python 
user_data[user_id]["state"] = "select_exercise"

@bot.command(name='progress')
async def progress(ctx):
```

## Visa användarens träningsprogress
```python 
await show_progress(ctx.message)

async def show_progress(message):
```

## Hämta användarens ID
```python 
user_id = message.author.id
```

## Skapa filnamnet för användarens träningsprogress
```python 
progress_file = f"{user_id}_progress.txt"
```

## Om filen för träningsprogressen finns
```python 
if os.path.exists(progress_file):
```

## Öppna filen och läs innehållet
```python 
with open(progress_file, "r") as file:
            progress_data = file.read()
```

## Skicka träningsprogressen till användaren
```python 
await message.author.send("Your workout progress:\n" + progress_data)
```

## Om ingen träningsprogress hittas, meddela användaren
```python 
else:
        await message.author.send("No recorded progress found. Please complete a workout first.")
```

# Starta botten
```python 
bot.run(TOKEN)
```