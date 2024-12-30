<script lang="ts">
	import { goto } from '$app/navigation';

	// Classes para coordenadas, status e inimigo
	class Coordenada {
		linha: number;
		coluna: number;
	}

	class Status {
		hp: number;
	}

	class EstadoJogo {
		posicaoPersonagem: Coordenada;
		posicaoObjetivo: Coordenada;
		posicaoEnemy: Coordenada;
		mapa: number[][];
	}

	// Inicializa o estado do jogo
	function inicializarJogo(): EstadoJogo {
		let personagem: Coordenada = new Coordenada();
		personagem.linha = 0;
		personagem.coluna = 0;

		let objetivo: Coordenada = new Coordenada();
		objetivo.linha = 9;
		objetivo.coluna = 9;

		let enemy: Coordenada = new Coordenada();
		enemy.linha = 5;
		enemy.coluna = 10;

		let mapa: number[][] = [
			[0, 0, 0, 0, 1, 0, 0, 0, 0, 0, 0, 0, 0, 0],
			[0, 0, 0, 0, 0, 0, 0, 0, 1, 0, 0, 0, 0, 0],
			[0, 1, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0],
			[0, 0, 0, 0, 0, 0, 1, 0, 0, 0, 0, 0, 0, 0],
			[0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0],
			[0, 0, 0, 0, 0, 0, 1, 0, 0, 0, 0, 0, 0, 0],
			[0, 0, 1, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0],
			[0, 0, 0, 0, 0, 0, 1, 0, 0, 0, 0, 0, 0, 0],
			[0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0],
			[0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 1],
		];

		let estado: EstadoJogo = new EstadoJogo();
		estado.posicaoPersonagem = personagem;
		estado.posicaoObjetivo = objetivo;
		estado.posicaoEnemy = enemy;
		estado.mapa = mapa;

		return estado;
	}

	// Movimenta o inimigo em direção ao jogador
	function enemyChase(posicao: Coordenada, jogador: Coordenada, mapa: number[][]): void {
		const deltaLinha = jogador.linha - posicao.linha;
		const deltaColuna = jogador.coluna - posicao.coluna;

		let novaLinha = posicao.linha;
		let novaColuna = posicao.coluna;

		// Movimenta na direção da menor distância
		if (Math.abs(deltaLinha) > Math.abs(deltaColuna)) {
			novaLinha += Math.sign(deltaLinha);
		} else {
			novaColuna += Math.sign(deltaColuna);
		}

		// Verifica se a nova posição é válida e não é a posição do jogador
		if (
			novaLinha >= 0 &&
			novaColuna >= 0 &&
			novaLinha < mapa.length &&
			novaColuna < mapa[0].length &&
			mapa[novaLinha][novaColuna] === 0 &&
			!(novaLinha === jogador.linha && novaColuna === jogador.coluna)
		) {
			posicao.linha = novaLinha;
			posicao.coluna = novaColuna;
		}
	}

	// Detecta colisão
	function houveColisao(posicao: Coordenada, jogo: EstadoJogo): boolean {
		return (
			posicao.linha < 0 ||
			posicao.coluna < 0 ||
			posicao.linha >= jogo.mapa.length ||
			posicao.coluna >= jogo.mapa[0].length ||
			jogo.mapa[posicao.linha][posicao.coluna] == 1
		);
	}

	// Controla os movimentos do jogador
	function onKeyDown(evento): void {
		let novaPosicao = new Coordenada();
		novaPosicao.linha = jogo.posicaoPersonagem.linha;
		novaPosicao.coluna = jogo.posicaoPersonagem.coluna;

		switch (evento.keyCode) {
			case 38: 
				novaPosicao.linha--;
				break;
			case 40: 
				novaPosicao.linha++;
				break;
			case 37: 
				novaPosicao.coluna--;
				break;
			case 39: 
				novaPosicao.coluna++;
				break;
		}

		// Detecta vitória
		if (
			novaPosicao.linha == jogo.posicaoObjetivo.linha &&
			novaPosicao.coluna == jogo.posicaoObjetivo.coluna
		) {
			alert('Parabéns, você chegou ao objetivo');
			goto('/');
		}

		// Detecta se o jogador colidiu com o inimigo
		if (
			novaPosicao.linha === jogo.posicaoEnemy.linha &&
			novaPosicao.coluna === jogo.posicaoEnemy.coluna
		) {
			alert('Você foi pego pelo inimigo!');
			goto('/');
			return;
		}

		// Movimenta o jogador
		if (!houveColisao(novaPosicao, jogo)) {
			jogo.posicaoPersonagem = novaPosicao;
		}

		// Movimenta o inimigo
		enemyChase(jogo.posicaoEnemy, novaPosicao, jogo.mapa);
	}

	let jogo: EstadoJogo = inicializarJogo();
</script>

<h1>Movimente o personagem (quadrado cinza) até o objetivo (quadrado roxo)</h1>

<table>
	{#each jogo.mapa as linha, i}
		<tr>
			{#each linha as celula, j}
				{#if i === jogo.posicaoPersonagem.linha && j === jogo.posicaoPersonagem.coluna}
					<td class="celula personagem"></td>
				{:else if i === jogo.posicaoObjetivo.linha && j === jogo.posicaoObjetivo.coluna}
					<td class="celula objetivo"></td>
				{:else if i === jogo.posicaoEnemy.linha && j === jogo.posicaoEnemy.coluna}
					<td class="celula enemy"></td>
				{:else if jogo.mapa[i][j] === 0}
					<td class="celula"></td>
				{:else}
					<td class="celula bloco"></td>
				{/if}
			{/each}
		</tr>
	{/each}
</table>

<a class="menu" href="/">Voltar ao Menu</a>

<svelte:window on:keydown|preventDefault={onKeyDown} />
