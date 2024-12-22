import pandas as pd
import matplotlib.pyplot as plt
import seaborn as sns
from fpdf import FPDF


file_path = "C:/Users/abdel/Downloads/data_real.csv"


def load_data(file_path):
    # Leer el archivo CSV
    data = pd.read_csv(file_path, delimiter=';')
    return data


def generate_plot(data, output_path="plot.png"):
    # Filtrar datos para tratamientos ABX y placebo
    filtered_data = data[data['treatment'].isin(['ABX', 'placebo'])]

   
    sns.set(style="whitegrid")
    plt.figure(figsize=(10, 6))
    sns.lineplot(
        data=filtered_data,
        x="experimental_day",
        y="frequency_live_bacteria_%",
        hue="treatment",
        marker="o"
    )

  
    plt.title("Frecuencia de bacterias vivas por día experimental", fontsize=14)
    plt.xlabel("Día experimental", fontsize=12)
    plt.ylabel("Frecuencia de bacterias vivas (%)", fontsize=12)
    plt.legend(title="Tratamiento")
    plt.tight_layout()

    
    plt.savefig(output_path)
    plt.close()


def create_pdf(data, plot_path, output_pdf="report.pdf"):
    pdf = FPDF()
    pdf.set_auto_page_break(auto=True, margin=15)
    pdf.add_page()

    
    pdf.set_font("Arial", size=16)
    pdf.cell(200, 10, txt="Informe de Datos Experimentales", ln=True, align='C')

   
    pdf.set_font("Arial", size=12)
    pdf.ln(10)
    pdf.multi_cell(0, 10, txt="Este informe contiene los datos experimentales de frecuencia de bacterias vivas, junto con un gráfico comparativo de los tratamientos.")

   
    pdf.ln(10)
    pdf.image(plot_path, x=30, y=None, w=150)

  
    pdf.add_page()
    pdf.set_font("Arial", size=12)
    pdf.cell(200, 10, txt="Vista de Datos (Primera Página)", ln=True, align='C')

  
    table_data = data.head(20).to_string(index=False)
    pdf.set_font("Courier", size=10)
    pdf.multi_cell(0, 10, txt=table_data)

    
    pdf.output(output_pdf)


def main():
   
    data = load_data(file_path)

    
    plot_path = "plot.png"
    generate_plot(data, plot_path)

   
    output_pdf = "report.pdf"
    create_pdf(data, plot_path, output_pdf)
    print(f"Informe PDF generado: {output_pdf}")

if __name__ == "__main__":
    main()
import pandas as pd
import seaborn as sns
import matplotlib.pyplot as plt

def create_violin_plot(file_path):
    """
    Crée un graphique en violon à partir d'un fichier Excel.
    
    :param file_path: Chemin vers le fichier Excel contenant les données
    """
    try:
        
        data = pd.read_csv(file_path, delimiter=';')

        
        sns.set(style="whitegrid")

        
        plt.figure(figsize=(10, 6))

       
        sns.violinplot(data=data, x="treatment", y="frequency_live_bacteria_%", palette="muted")

        
        plt.title("Violin Plot of Frequency of Live Bacteria by Treatment", fontsize=14)
        plt.xlabel("Treatment", fontsize=12)
        plt.ylabel("Frequency of Live Bacteria (%)", fontsize=12)
        plt.xticks(fontsize=10)
        plt.yticks(fontsize=10)

      
        plt.tight_layout()
        plt.show()

    except Exception as e:
        print(f"Une erreur s'est produite : {e}")


file_path = "data_real.csv"


create_violin_plot(file_path)

import pandas as pd
import matplotlib.pyplot as plt
import seaborn as sns
import os


os.makedirs('input', exist_ok=True)
os.makedirs('images', exist_ok=True)
os.makedirs('output', exist_ok=True)


SIZE_CATEGORIES = {
    "small": 0,        # Taille <= 1 000 lignes
    "medium": 1000,    # Taille > 1 000 et <= 10 000 lignes
    "large": 10000,    # Taille > 10 000 et <= 100 000 lignes
    "huge": 100000     # Taille > 100 000 lignes
}

def categorize_file_size(file_path):
    """
    Catégorise un fichier CSV en fonction de son nombre de lignes.
    """
    num_lines = sum(1 for line in open(file_path)) - 1  # Compte les lignes, sans l'en-tête
    if num_lines <= SIZE_CATEGORIES["medium"]:
        return "small"
    elif num_lines <= SIZE_CATEGORIES["large"]:
        return "medium"
    elif num_lines <= SIZE_CATEGORIES["huge"]:
        return "large"
    else:
        return "huge"


input_file = 'input/data.csv'
size_category = categorize_file_size(input_file)
print(f"Le fichier est catégorisé comme '{size_category}' en fonction de sa taille.")

data = pd.read_csv(input_file)


fecal_data = data[data['sample_type'] == 'fécal']
for mouse_id in fecal_data['mouse_id'].unique():
    mouse_data = fecal_data[fecal_data['mouse_id'] == mouse_id]
    plt.plot(mouse_data['experimental_day'], mouse_data['counts_live_bacteria'], label=mouse_id)

plt.title(f'Quantité de bactéries fécales par souris ({size_category})')
plt.xlabel('Jour expérimental')
plt.ylabel('Quantité de bactéries vivantes')
plt.legend()
plt.savefig(f'images/graphique_fecal_{size_category}.png')
plt.close()


caecal_data = data[data['sample_type'] == 'cécal']
sns.violinplot(x='treatment', y='counts_live_bacteria', data=caecal_data)
plt.title(f'Dispersion des bactéries (cécal) - {size_category}')
plt.xlabel('Traitement')
plt.ylabel('Quantité de bactéries vivantes')
plt.savefig(f'images/graphique_caecal_violin_{size_category}.png')
plt.close()


fecal_data.to_csv(f'output/fecal_data_{size_category}.csv', index=False)
caecal_data.to_csv(f'output/caecal_data_{size_category}.csv', index=False)

print(f"Traitement terminé pour le fichier '{size_category}'. Vérifiez les dossiers output/ et images/")
