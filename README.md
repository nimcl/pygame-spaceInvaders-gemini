# pygame-spaceInvaders-gemini
Réplica de space invaders com feedback gerado por IA




	import pygame
	import random
	import time
	import os
	import uuid
	import google.generativeai as genai

	pygame.init()

	x = 1280
	y = 720
	fonte = pygame.font.SysFont('arial', 50)
	fonte_pequena = pygame.font.SysFont('arial', 30)
	logado = False

	cenario = pygame.Surface((x, y))
	cenario.fill((255, 255, 255))

	inimigo = pygame.Surface((100, 70))
	inimigo.fill((255, 0, 0))

	jogador = pygame.Surface((200, 200))
	jogador.fill((0, 0, 255))

	tiro = pygame.Surface((50, 20))
	tiro.fill((0, 255, 0))

	jogador_x = 100
	jogador_y = 300
	tiro_x = 100
	tiro_y = 300
	inimigo_x = 500
	inimigo_y = 360
	pontos = 3
	inimigos_derrotados = 0
	disparo = False
	velocidade_x_tiro = 0
	tempo_inicio = time.time()
	numero_jogada = 1

	tela = pygame.display.set_mode((x, y))
	pygame.display.set_caption('Joguinho Python')

	cenario = pygame.image.load('imagens/cenario.jpg').convert_alpha()
	cenario = pygame.transform.scale(cenario, (x, y))

	inimigo = pygame.image.load('imagens/inimigo.png').convert_alpha()
	inimigo = pygame.transform.scale(inimigo, (100, 70))

	jogador = pygame.image.load('imagens/jogador.png').convert_alpha()
	jogador = pygame.transform.scale(jogador, (200, 200))

	tiro = pygame.image.load('imagens/missil.png').convert_alpha()
	tiro = pygame.transform.scale(tiro, (50, 20))

	email_rect = pygame.Rect(50, 50, 400, 50)
	senha_rect = pygame.Rect(50, 100, 400, 50)
	botao_rect = pygame.Rect(50, 200, 100, 50)
	email_active = False
	senha_active = False
	email = ""
	senha = ""
	mensagem_erro = ""

	numero_jogada = 1

	def criar_diretorio(diretorio_usuario):
    	diretorio_completo = os.path.join(diretorio_usuario, 'dados_jogadas')
    	os.makedirs(diretorio_completo, exist_ok=True)
    	return diretorio_completo

	def salvar_dados_jogador(diretorio_jogadas, email, pontos, inimigos_derrotados, tempo_jogo, numero_jogada):
    	nome_arquivo = f'{uuid.uuid4()}.txt'
   	  caminho_arquivo = os.path.join(diretorio_jogadas, nome_arquivo)
    	with open(caminho_arquivo, 'w') as file:
        	file.write(f'Email: {email}\n')
        	file.write(f'Pontos: {pontos}\n')
        	file.write(f'Inimigos Derrotados: {inimigos_derrotados}\n')
        	file.write(f'Tempo de Jogo: {tempo_jogo:.2f} segundos\n')

	def tela_login():
    	global email, senha, logado, mensagem_erro, email_active, senha_active
    	email_active = False
    	senha_active = False
    	numero_jogada = 1  
    	while not logado:
        	for event in pygame.event.get():
            	if event.type == pygame.QUIT:
                	pygame.quit()
                	exit()
            	if event.type == pygame.KEYDOWN:
                	if event.key == pygame.K_BACKSPACE:
                    	if senha_active:
                        	senha = senha[:-1]
                    	else:
                        	email = email[:-1]
                	else:
                    	if senha_active:
                        	senha += event.unicode
                    	else:
                        	email += event.unicode
            	if event.type == pygame.MOUSEBUTTONDOWN:
                	if email_rect.collidepoint(event.pos):
                    	email_active = True
                    	senha_active = False
                	elif senha_rect.collidepoint(event.pos):
                    	senha_active = True
                    	email_active = False
                	elif botao_rect.collidepoint(event.pos):
                    	if email == "Teste" and senha == "Teste":
                        	logado = True
                    	else:
                        	mensagem_erro = "Email ou senha incorretos"

        	tela.fill((255, 255, 255))

        	email_surface = fonte_pequena.render(f'Email: {email}', True, (0, 0, 0))
        	senha_surface = fonte_pequena.render(f'Senha: {"*" * len(senha)}', True, (0, 0, 0))
        	mensagem_surface = fonte_pequena.render(mensagem_erro, True, (255, 0, 0))

        	tela.blit(email_surface, (email_rect.x + 5, email_rect.y + 5))
        	tela.blit(senha_surface, (senha_rect.x + 5, senha_rect.y + 5))
        	tela.blit(mensagem_surface, (50, 150))

        	pygame.draw.rect(tela, (0, 0, 0), email_rect, 2)
        	pygame.draw.rect(tela, (0, 0, 0), senha_rect, 2)
        	pygame.draw.rect(tela, (0, 0, 0), botao_rect, 2)

        	botao_surface = fonte_pequena.render('Login', True, (0, 0, 0))
        	tela.blit(botao_surface, (botao_rect.x + 20, botao_rect.y + 10))

        	pygame.display.flip()

    	main(numero_jogada)  


	def voltar():
    	x = 1350
    	y = random.randint(1, 640)
    	return [x, y]

	def voltar_tiro():
    	global disparo, velocidade_x_tiro
    	disparo = False
    	voltar_tiro_x = jogador_x + 150 
    	voltar_tiro_y = jogador_y + 90
    	velocidade_x_tiro = 0
    	return [voltar_tiro_x, voltar_tiro_y]

	def colisao(jogador_rect, inimigo_rect, tiro_rect):
    	global pontos, inimigos_derrotados
    	if jogador_rect.colliderect(inimigo_rect):
        	pontos -= 1
        	return True
    	elif tiro_rect.colliderect(inimigo_rect):
        	pontos += 1
        	inimigos_derrotados += 1
        	return True
    	return False

	def mostrar_relatorio(diretorio_dados):  
    	global pontos, inimigos_derrotados, tempo_jogo, numero_jogada
    	tela.fill((255, 255, 255))
    	relatorio = [
        	f"Email: {email}",
        	f"Pontos: {pontos}",
        	f"Inimigos Derrotados: {inimigos_derrotados}",
        	f"Tempo de Jogo: {tempo_jogo:.2f} segundos"
    	]
    	y_offset = 100
    	for linha in relatorio:
        	texto_surface = fonte.render(linha, True, (0, 0, 0))
        	tela.blit(texto_surface, (50, y_offset))
        	y_offset += 60
    	pygame.display.flip()
    	pygame.time.wait(5000)  # Espera 5 segundos antes de fechar
    	reiniciar_jogo(numero_jogada, diretorio_dados)  


	def reiniciar_jogo(numero_jogada, diretorio_dados):
    	global jogador_x, jogador_y, tiro_x, tiro_y, inimigo_x, inimigo_y, pontos, inimigos_derrotados, tempo_inicio, logado, disparo
    	numero_jogada += 1  
    	jogador_x = 100
    	jogador_y = 300
    	tiro_x = 100
    	tiro_y = 300
    	inimigo_x = 500
    	inimigo_y = 360
    	pontos = 3
    	inimigos_derrotados = 0
    	tempo_inicio = time.time()
    	logado = False  
    	disparo = False  
    	salvar_dados_jogador(diretorio_dados, email, pontos, inimigos_derrotados, tempo_jogo, numero_jogada)  
    	tela_login()  


	def main(numero_jogada):  
    	global rodando, tempo_fim, jogador_x, jogador_y, tiro_x, tiro_y, inimigo_x, inimigo_y, pontos, inimigos_derrotados, tempo_inicio, velocidade_x_tiro, disparo  
    	global disparo, tempo_jogo  
    	diretorio_dados = criar_diretorio('C:\\Users\\nicol\\OneDrive\\Área de Trabalho\\codigos_py\\imagens\\dados_do_usuario')
    	rodando = True
    	tempo_inicio = time.time()
    	velocidade_x_tiro = 0  

    	jogador_rect = jogador.get_rect()
    	inimigo_rect = inimigo.get_rect()
    	tiro_rect = tiro.get_rect()

    	while rodando:
        	for event in pygame.event.get():
            	if event.type == pygame.QUIT:
                	rodando = False
            	if event.type == pygame.KEYDOWN:
                	if event.key == pygame.K_x:
                    	rodando = False

        	tela.blit(cenario, (0, 0))
        	tela.blit(inimigo, (inimigo_x, inimigo_y))
        	tela.blit(tiro, (tiro_x, tiro_y))
        	tela.blit(jogador, (jogador_x, jogador_y))

        	tecla = pygame.key.get_pressed()
        	if tecla[pygame.K_UP] and jogador_y > 1:
            	jogador_y -= 1
            	if not disparo:
                	tiro_y -= 1
        	if tecla[pygame.K_DOWN] and jogador_y < 520:
            	jogador_y += 1
            	if not disparo:
                	tiro_y += 1
        	if tecla[pygame.K_SPACE] and not disparo:
            	disparo = True
            	velocidade_x_tiro = 3
            	tiro_x = jogador_x

        	if pontos < 1:
            	rodando = False

        	if inimigo_x < 0:
            	inimigo_x, inimigo_y = voltar()

        	if tiro_x > 1300:
            	tiro_x, tiro_y = voltar_tiro()

        	if colisao(jogador_rect, inimigo_rect, tiro_rect):
            	inimigo_x, inimigo_y = voltar()

        	jogador_rect.topleft = (jogador_x, jogador_y)
        	tiro_rect.topleft = (tiro_x, tiro_y)
        	inimigo_rect.topleft = (inimigo_x, inimigo_y)

        	tiro_x += velocidade_x_tiro
        	inimigo_x -= 1

        	pygame.draw.rect(tela, (255, 0, 0), jogador_rect, 2)
        	pygame.draw.rect(tela, (255, 0, 0), tiro_rect, 2)
        	pygame.draw.rect(tela, (255, 0, 0), inimigo_rect, 2)

        	score = fonte.render(f'Pontos:{int(pontos)}', True, (0, 0, 0))
        	tela.blit(score, (50, 50))

        	pygame.display.update()
        
        	jogador_rect.x = jogador_x
        	jogador_rect.y = jogador_y
        	tiro_rect.x = tiro_x
        	tiro_rect.y = tiro_y
        	inimigo_rect.x = inimigo_x
        	inimigo_rect.y = inimigo_y
     
        	tiro_x += velocidade_x_tiro
        	inimigo_x -=2
    
    	tempo_fim = time.time()
    	tempo_jogo = tempo_fim - tempo_inicio

    	salvar_dados_jogador(diretorio_dados, email, pontos, inimigos_derrotados, tempo_jogo, numero_jogada)
    	mostrar_relatorio(diretorio_dados)  


	genai.configure(api_key="AIzaSyAnD4dhN3Of5E6f_FndVOnmzxaNyzhn1s4")
	model = genai.GenerativeModel('gemini-pro')

	diretorio_resultados = r"C:\Users\nicol\OneDrive\Área de Trabalho\codigos_py\imagens\dados_do_usuario\dados_jogadas"
	arquivos = os.listdir(diretorio_resultados)

	def ler_arquivo_txt(caminho_arquivo):
    	with open(caminho_arquivo, 'r') as file:
        	linhas = file.readlines()
        	email = linhas[0].split(":")[1].strip()
        	pontos = int(linhas[1].split(":")[1].strip())
        	inimigos_derrotados = int(linhas[2].split(":")[1].strip())
        	tempo_jogo = float(linhas[3].split(":")[1].split()[0].strip())
        	return email, pontos, inimigos_derrotados, tempo_jogo


	dados_partidas = []

	for arquivo in arquivos:
    	if arquivo.endswith(".txt"):
        	caminho_arquivo = os.path.join(diretorio_resultados, arquivo)
        	dados_partida = ler_arquivo_txt(caminho_arquivo)
        	dados_partidas.append(dados_partida)

	resumo = []
	for email, pontos, inimigos_derrotados, tempo_jogo in dados_partidas:
    	resumo.append(f"Jogador: {email}")
    	resumo.append(f"Pontos: {pontos}")
    	resumo.append(f"Inimigos Derrotados: {inimigos_derrotados}")
    	resumo.append(f"Tempo de Jogo: {tempo_jogo:.2f} segundos")
    	resumo.append("\n")

	 response = model.generate_content(
    resumo + ["[END]\n\nFaça um feedback dizendo onde o jogador pode melhorar, e com base nos dados diga se o jogador obteve melhora"]
	)

	print(response.text)

	tela_login()
