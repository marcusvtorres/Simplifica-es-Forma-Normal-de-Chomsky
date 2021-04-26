# Simplifica-es-Forma-Normal-de-Chomsky
	from string import letras
	import copy
	import re
	

	# Deleta regras longas (mais que dois estados a direita ex.: A->BCD)
	def longo(regras,let,voc):
	

	    
	
	    new_dict = copy.deepcopy(regras)
	    for key in new_dict:
	        values = new_dict[key]
	        for i in range(len(values)):
                    
	            
	            if len(values[i]) > 2:
	

	                # A -> BCD se dá 1) A-> BE (se E é a primeira letra "livre"
	                # do Repositório de letras) e  2) E-> CD
	                for j in range(0, len(values[i]) - 2):
                            
	                    # substitui 1 regra
	                    if j==0:
	                        regras[key][i] = regras[key][i][0] + let[0]
	                        
	                    # add novas regras
	                    else:
	                        regras.setdefault(new_key, []).append(values[i][j] + let[0])
	                        voc.append(let[0])
	                        
	                    # salva letra, que será usada na próxima regra
	                    new_key = copy.deepcopy(let[0])
	                    
	                    # remove letra da lista de letras livres
	                    let.remove(let[0])
	                    
	                # 2 ultimas letras sao sempre as mesmas
	                regras.setdefault(new_key, []).append(values[i][-2:])
	

	    return regras,let,voc
	

	

	# Deletar regras vazias (A->e)
	def empty(regras,voc):
	

	    # lista com keys de regras vazias
	    e_list = []
	

	    # Encontrar regras não terminais e adicionar na lista
	    new_dict = copy.deepcopy(regras)
	    for key in new_dict:
	        values = new_dict[key]
	        for i in range(len(values)):
                    
	            # se a key dá estado vazio e não está na lista, o adiciona
	            if values[i] == 'e' and key not in e_list:
	                e_list.append(key)
	                
	                # Remover estado VAZIO
	                regras[key].remove(values[i])
	                
	        # se key não contem nenhum valor, remove do dicionario
	        if len(regras[key]) == 0:
	            if key not in regras:
	                voc.remove(key)
	            regras.pop(key, None)
	

	    # Remoções de produçoes Vazias
	    new_dict = copy.deepcopy(regras)
	    for key in new_dict:
	        values = new_dict[key]
	        for i in range(len(values)):
	            # verificar regras na forma A->BC ou A->CB, onde B está lista e C no vocabulário
	
	            if len(values[i]) == 2:
	                # verifica regra na forma A->BC, excluindo o caso que #resulta A->A

	                if values[i][0] in e_list and key!=values[i][1]:
	                    regras.setdefault(key, []).append(values[i][1])
	                # verifica regra na forma A->CB, excluindo caso que resulta #A->A

	                if values[i][1] in e_list and key!=values[i][0]:
	                    if values[i][0]!=values[i][1]:
	                        regras.setdefault(key, []).append(values[i][0])
	

	    return regras,voc
	

	# Remove regras curtas (A->B)
	def short(regras,voc):
	

	    # cria um dicionário na forma letra:letra (no início D(A) = {A})
	
	    D = dict(zip(voc, voc))
	

	    # transforma valor da string para lista para poder inserir mais valores
	    for key in D:
	        D[key] = list(D[key])
	

	    # para cada letra A do vocabulário, se B->C, B em D(A) e C não em D(A) adiciona C em D(A)
	    for letter in voc:
	        for key in regras:
	            if key in D[letter]:
	                values = regras[key]
	                for i in range(len(values)):
	                    if len(values[i]) == 1 and values[i] not in D[letter]:
	                        D.setdefault(letter, []).append(values[i])
	

	    regras,D = short1(regras,D)
	    return regras,D
	

	

	def short1(regras,D):
	

	    # remove regras curtas (com o tamanho a direita = 1)
	    new_dict = copy.deepcopy(regras)
	    for key in new_dict:
	        values = new_dict[key]
	        for i in range(len(values)):
	            if len(values[i]) == 1:
	                regras[key].remove(values[i])
	        if len(regras[key]) == 0: regras.pop(key, None)
	

	    # substitui cada regra A->BC com A->B'C', onde B' em D(B) e C' em D(C)
	    for key in regras:
	        values = regras[key]
	        for i in range(len(values)):
	            # procura todo possivel B' em D(B)
	            for j in D[values[i][0]]:
	                # procura todo possC' em D(C)
	                for k in D[values[i][1]]:
	                    # concatena B' e C' e insere nova regra
	                    if j+k not in values:
	                        regras.setdefault(key, []).append(j + k)
	

	    return regras,D
	

	

	# Insere regra S->BC para cada A->BC onde A em D(S)-{S}
	def final_regras(regras,D,S):
	

	    for let in D[S]:
	        # verifca se key não tem nenhum valor
	        if not regras[S] and not regras[let]:
	            for v in regras[let]:
	                if v not in regras[S]:
	                    regras.setdefault(S, []).append(v)
	    return regras
	

	# Printa regras
	def print_regras(regras):
	    for key in regras:
	        values = regras[key]
	        for i in range(len(values)):
	            print key + '->' + values[i]
	    return 1
	

	

	def main():
	

	    regras = {}
	    voc = []
	    # Essa lista será nosso "repositório de letras" para nomear estados
	    let = list(letras[26:]) + list(letras[:25])
	

	    let.remove('e')
	

	    # Número de regras gramaticais
	    while True:
	        userInput = raw_input('Digite o número de regras')
	        try:
	            # verifica se N é inteiro >=2
	            N = int(userInput)
	            if N <=2: print 'N deve ser um número >=2!'
	            else: break
	        except ValueError:
	            print "Não é um número inteiro!"
	

	    # Estado inicial
	    while True:
	        S = raw_input('Digite um estado inicial')
	        if not re.match("[a-zA-Z]*$", S): print 'Estado inicial deve ser um único caractere!'
	
	        else:break
	

	    print '+------------------------------------------------------+'
	    print '|Digite as regras na forma A B (com espaço), para A->B |'
	    print '|ou  A BCD, se mais de um estado está a direita        |'
	    print '|(sem espaços entre os itens a direita).               |'
	    print '+------------------------------------------------------+'
	

	    for i in range(N):
	        fr, to = map(str,raw_input('Rule #' + str(i + 1)).split())
	        # Remover letras inseridas do "Repositório de letras"
	        for l in fr:
	            if l!='e' and l not in voc: voc.append(l)
	            if l in let: let.remove(l)
	        for l in to:
	            if l!='e' and l not in voc: voc.append(l)
	            if l in let: let.remove(l)
	        # Inserir regra ao dicionário
	        regras.setdefault(fr, []).append(to)
	

	    # remove regras Longas printa novas regras
	    print '\nRegras após remoção L'
	    regras,let,voc = longo(regras,let,voc)
	    print_regras(regras)
	    #print voc
	

	    # remove regras vazias printa novas regras
	    print '\nRegras após remoção dos vazios'
	    regras,voc = empty(regras,voc)
	    print_regras(regras)
	    #print voc
	

	    print '\nRegras apos remoção das curtas'
	    regras,D = short(regras,voc)
	    print_regras(regras)
	

	    print '\nRegras finais'
	    regras = final_regras(regras,D,S)
	    print_regras(regras)
	

	if __name__ == '__main__':
	    main()
