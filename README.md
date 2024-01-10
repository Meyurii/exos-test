import tkinter as tk
import random

class Pion:
    def __init__(self, joueur, coord=None):
        self.joueur = joueur
        self.coord = coord

class Jeu:
    def __init__(self, dimension=10, pions_aligner=5):
        self.dimension = dimension
        self.pions_aligner = pions_aligner
        self.plateau = [['' for _ in range(dimension)] for _ in range(dimension)]
        self.joueurs = ['Rouge', 'Bleu']
        self.joueur_actuel = random.choice(self.joueurs)
        self.pions = {'Rouge': Pion('Rouge', None), 'Bleu': Pion('Bleu', None)}
        self.creer_interface()

    def creer_interface(self):
        self.root = tk.Tk()
        self.root.title("Jeu de Réflexion et Logique")

        self.canvas = tk.Canvas(self.root, width=600, height=600)
        self.canvas.pack()

        self.canvas.bind("<Button-1>", self.gerer_clic)

        self.afficher_plateau()

        self.root.mainloop()

    def afficher_plateau(self):
        self.canvas.delete("all")

        for i in range(self.dimension):
            for j in range(self.dimension):
                x0, y0, x1, y1 = j * 60, i * 60, (j + 1) * 60, (i + 1) * 60
                self.canvas.create_rectangle(x0, y0, x1, y1, outline="black")

                if self.plateau[i][j] == 'Rouge':
                    if self.pions['Rouge'].coord == (i, j):
                        self.canvas.create_oval(x0 + 10, y0 + 10, x1 - 10, y1 - 10, fill="#FF0000", outline="#FF0000")
                    else:
                        self.afficher_cross(i, j, "#FF0000")

                elif self.plateau[i][j] == 'Bleu':
                    if self.pions['Bleu'].coord == (i, j):
                        self.canvas.create_oval(x0 + 10, y0 + 10, x1 - 10, y1 - 10, fill="#0000FF", outline="#0000FF")
                    else:
                        self.afficher_cross(i, j, "#0000FF")

        self.root.title(f"Tour de {self.joueur_actuel}")
        self.previsualiser_coups()

    def previsualiser_coups(self):
        if self.pions[self.joueur_actuel].coord is not None:
            for move in self.get_valid_moves(self.pions[self.joueur_actuel].coord):
                x0, y0, x1, y1 = move[1] * 60 + 10, move[0] * 60 + 10, (move[1] + 1) * 60 - 10, (move[0] + 1) * 60 - 10
                self.canvas.create_oval(x0, y0, x1, y1, fill="gray", outline="gray")

    def gerer_clic(self, event):
        colonne = event.x // 60
        ligne = event.y // 60

        if self.plateau[ligne][colonne] == '' and self.deplacement_valide(self.pions[self.joueur_actuel].coord, (ligne, colonne)):
            # Check if the pawn is moving from a square
            if self.pions[self.joueur_actuel].coord:
                ancienne_ligne, ancienne_colonne = self.pions[self.joueur_actuel].coord

                # Transform the pawn into a cross in the previous position
                self.afficher_cross(ancienne_ligne, ancienne_colonne, self.get_color(self.joueur_actuel))

            self.plateau[ligne][colonne] = self.joueur_actuel
            self.pions[self.joueur_actuel].coord = (ligne, colonne)

            if self.verifier_victoire():
                self.afficher_resultat()
            elif self.verifier_blocage():
                self.afficher_resultat_blocage()
            else:
                self.joueur_suivant()
                self.afficher_plateau()

    def afficher_cross(self, ligne, colonne, couleur):
        x0, y0, x1, y1 = colonne * 60, ligne * 60, (colonne + 1) * 60, (ligne + 1) * 60
        self.canvas.create_line(x0, y0, x1, y1, fill=couleur, width=2)
        self.canvas.create_line(x0, y1, x1, y0, fill=couleur, width=2)

    def verifier_blocage(self):
        for i in range(self.dimension):
            for j in range(self.dimension):
                if self.plateau[i][j] == '' or self.plateau[i][j] == self.joueur_actuel:
                    return False

        return True

    def afficher_resultat_blocage(self):
        self.canvas.delete("all")
        self.canvas.create_text(300, 300, text=f"{self.joueur_actuel} a gagné par blocage!", font=('Helvetica', 18, 'bold'))
        self.root.title(f"{self.joueur_actuel} a gagné par blocage!")

    def joueur_suivant(self):
        self.joueur_actuel = [j for j in self.joueurs if j != self.joueur_actuel][0]

    def deplacement_valide(self, coord_actuelle, nouvelle_coord):
        if coord_actuelle is None:
            return True

        dx = abs(nouvelle_coord[0] - coord_actuelle[0])
        dy = abs(nouvelle_coord[1] - coord_actuelle[1])

        return (dx == 1 and dy == 2) or (dx == 2 and dy == 1) and self.plateau[nouvelle_coord[0]][nouvelle_coord[1]] == ''

    def verifier_victoire(self):
        ligne, colonne = self.pions[self.joueur_actuel].coord

        # Vérifier l'alignement horizontal
        if self.verifier_alignement(self.plateau[ligne]):
            return True

        # Vérifier l'alignement vertical
        if self.verifier_alignement([self.plateau[i][colonne] for i in range(self.dimension)]):
            return True

        # Vérifier l'alignement diagonal
        if ligne == colonne:
            if self.verifier_alignement([self.plateau[i][i] for i in range(self.dimension)]):
                return True

        if ligne + colonne == self.dimension - 1:
            if self.verifier_alignement([self.plateau[i][self.dimension - 1 - i] for i in range(self.dimension)]):
                return True

        return False

    def verifier_alignement(self, ligne_colonne_diagonale):
        return ''.join(ligne_colonne_diagonale).count(self.joueur_actuel) >= self.pions_aligner

    def afficher_resultat(self):
        self.canvas.delete("all")
        self.canvas.create_text(300, 300, text=f"{self.joueur_actuel} a gagné!", font=('Helvetica', 18, 'bold'))
        self.root.title(f"{self.joueur_actuel} a gagné!")

    def get_valid_moves(self, coord):
        moves = []
        for i in range(-2, 3):
            for j in range(-2, 3):
                new_coord = (coord[0] + i, coord[1] + j)
                if 0 <= new_coord[0] < self.dimension and 0 <= new_coord[1] < self.dimension:
                    if self.deplacement_valide(coord, new_coord) and self.plateau[new_coord[0]][new_coord[1]] == '':
                        moves.append(new_coord)
        return moves

    def get_color(self, joueur):
        return "#FF0000" if joueur == 'Rouge' else "#0000FF"

if __name__ == "__main__":
    jeu = Jeu()

