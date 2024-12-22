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

