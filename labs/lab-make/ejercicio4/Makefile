# Nombre del ejecutable final
PROGRAMA := programa

# Compilador y flags
CC     := gcc
CFLAGS := -Wall

SRCS := main.c operaciones.c


OBJS := $(SRCS:.c=.o)

# Regla principal
all: $(PROGRAMA)

$(PROGRAMA): $(OBJS)
	$(CC) $^ -o $(PROGRAMA)
	$(CC) $(CFLAGS) -c $< -o $@

# Regla para limpiar archivos generados
clean:
	rm -f $(PROGRAMA) $(OBJS)
	
.PHONY: all clean