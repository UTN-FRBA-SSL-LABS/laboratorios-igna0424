# Nombre del ejecutable final
PROGRAMA := calc3

# Compilador
CC := gcc

# Regla principal
all: $(PROGRAMA)

# Regla para construir la calculadora a partir del parser y el scanner
# Bison genera: parser3.tab.c y parser3.tab.h
# Flex genera:  lex.yy.c
$(PROGRAMA): parser3.y scanner3.l
	bison -d parser3.y

	flex scanner3.l

	$(CC) lex.yy.c parser3.tab.c -o $(PROGRAMA)
	

# Regla para limpiar todos los archivos generados
clean:
	rm -f $(PROGRAMA)
	rm -f lex.yy.c
	rm -f parser3.tab.c
	rm -f parser3.tab.h
.PHONY: all clean
