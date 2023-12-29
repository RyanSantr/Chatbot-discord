# CAHT DE iventario, modelo manual

import discord
from discord.ext import commands
import sqlite3
import asyncio

intents = discord.Intents.default()  # Create Intents object
bot = commands.Bot(command_prefix='!',
                   intents=intents)  # Pass Intents object to Bot instantiation


class PecaRoupa:

  def __init__(self, nome, quantidade, material, etapa, tempo):
    self.nome = nome
    self.quantidade = quantidade
    self.material = material
    self.etapa = etapa
    self.tempo = tempo

  def __str__(self):
    return f"Peça: {self.nome}, Quantidade: {self.quantidade}, Material: {self.material}, Etapa: {self.etapa}, Tempo: {self.tempo}"


def inserir_peca_no_banco(peca):
  conn = sqlite3.connect('inventario.db')
  cursor = conn.cursor()
  cursor.execute('''CREATE TABLE IF NOT EXISTS pecas (
                        id INTEGER PRIMARY KEY,
                        nome TEXT,
                        quantidade INTEGER,
                        material TEXT,
                        etapa TEXT,
                        tempo INTEGER)''')
  cursor.execute(
      "INSERT INTO pecas (nome, quantidade, material, etapa, tempo) VALUES (?, ?, ?, ?, ?)",
      (peca.nome, peca.quantidade, peca.material, peca.etapa, peca.tempo))
  conn.commit()
  conn.close()


def recuperar_pecas_do_banco():
  conn = sqlite3.connect('inventario.db')
  cursor = conn.cursor()
  cursor.execute("SELECT * FROM pecas")
  pecas = []
  for row in cursor.fetchall():
    peca = PecaRoupa(row[1], row[2], row[3], row[4], row[5])
    pecas.append(peca)
  conn.close()
  return pecas


@bot.event
async def on_ready():
  print(f'Bot conectado como {bot.user.name}')


@bot.command()
async def adicionar(ctx, nome, quantidade, material, etapa, tempo):
  peca = PecaRoupa(nome, quantidade, material, etapa, tempo)
  inserir_peca_no_banco(peca)
  await ctx.send(
      f" `Peça de roupa '{nome}' adicionada ao inventário. O timer será acionado em {tempo} segundos.`"
  )
  await asyncio.sleep(int(tempo))  # Add a delay here
  await ctx.send(f"`O temporizador para a peça de roupa '{nome}' acabou!`")


@bot.command()
async def listar(ctx):
  pecas = recuperar_pecas_do_banco()
  text = "` Inventário de Peças de Roupa:\n ` "
  for peca in pecas:
    text += str(peca) + "` \n `"
  await ctx.send(text)


bot.run(
    'Seu Token')
