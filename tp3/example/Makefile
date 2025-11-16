# L'exécutable à produire
TARGET = Test
# La commande de compilation
CC = gcc
# Arguments pour la compilation
CCFLAGS = -c -Wall
# Arguments pour le linking
COFLAGS = -o

# Les fichiers à utiliser
SRCFILES = $(wildcard *.c)
HDRFILES = $(wildcard *.h)
OBJFILES = $(patsubst %.c, build/%.o, $(SRCFILES))

# La target .PHONY indique les cibles "virtuelles", qui n'existent pas réellement
.PHONY: all clean run

# Par convention, on crée une cible all pour `make all`. C'est la première cible, donc la default
all: $(TARGET)

# Une cible virtuelle de nettoyage
clean:
	-rm -rf build
	-rm -rf $(TARGET)

run: all
	./$(TARGET)

# Création de la cible pour les .o, dépendant des sources
$(OBJFILES): $(SRCFILES) $(HDRFILES)
	-mkdir build
	$(CC) $(CCFLAGS) $(SRCFILES)
	mv *.o build/

# Création de la cible pour l'exécutable, dépendant des .o
$(TARGET): $(OBJFILES)
	$(CC) $(COFLAGS) $(TARGET) $(OBJFILES)
