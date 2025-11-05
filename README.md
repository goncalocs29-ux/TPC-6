# TPC-6
from datetime import date, datetime
import json
import os
from typing import List, Tuple, Optional

try:
    import matplotlib.pyplot as plt
except ImportError:
    plt = None

# Exemplo de dados
TabMeteo = List[Tuple[Tuple[int, int, int], float, float, float]]

tabMeteo1: TabMeteo = [((2022, 1, 20), 2.0, 16.0, 0.0), ((2022, 1, 21), 1.0, 13.0, 0.2), ((2022, 1, 22), 7.0, 17.0, 0.01),]

#1.a) Calcula a temperatura média de cada dia (lista de pares: [(data, temperaturaMédia)])
def medias(tabMeteo):
    res = []
    for reg in tabMeteo:
        data, tmin, tmax, prec = reg
        media = (tmin + tmax) / 2
        res.append((data, media))
    return res

#1.b) Define uma função para guardar uma tabela meteorológica num ficheiro JSON
def guardaTabMeteo(t: TabMeteo, fnome: str) -> None:
    json_list = []
    for (y, m, d), tmin, tmax, prec in t:
        json_list.append({
            "date": f"{y:04d}-{m:02d}-{d:02d}",
            "tmin": float(tmin),
            "tmax": float(tmax),
            "prec": float(prec)
        })
    with open(fnome, "w", encoding="utf-8") as f:
        json.dump(json_list, f, indent=2)

 #1.c) Define uma função para carregar uma tabela meteorológica de um ficheiro de texto. Carrega uma tabela de ficheiro JSON
def carregaTabMeteo(fnome):
    res = []
    if not os.path.exists(fnome):
        return res
    with open(fnome, "r", encoding="utf-8") as f:
        data = json.load(f)
        for item in data:
            try:
                # Converte a string "YYYY-MM-DD" para (ano, mês, dia)
                dt = datetime.strptime(item["date"], "%Y-%m-%d").date()
                data_tuplo = (dt.year, dt.month, dt.day)
                tmin = float(item["tmin"])
                tmax = float(item["tmax"])
                prec = float(item["prec"])

                res.append((data_tuplo, tmin, tmax, prec))
            except:
                continue
    return res

tabMeteo2 = carregaTabMeteo("meteorologia.json")
print(tabMeteo2)

#1.d) Calcula a temperatura mínima mais baixa registada na tabela, dando como resultado esse valor
def minMin(tabMeteo):
    if not tabMeteo:   # Se a lista estiver vazia
        return None

    minima = tabMeteo[0][1]

    for (data, tmin, tmax, prec) in tabMeteo:
        if tmin < minima:
            minima = tmin

    return minima

print(minMin(tabMeteo1))

# 1.e) Calcula a amplitude térmica (diferença entre a temperatura máxima e a temperatura mínima) de cada dia, dando como resultado uma lista de pares: [(data, amplitude)]
def amplTerm(tabMeteo):
    res = []
    for (data, tmin, tmax, prec) in tabMeteo:
        amplitude = tmax - tmin
        res.append((data, amplitude))
    return res

print(amplTerm(tabMeteo1))

#1.f) Calcula o dia em que a precipitação registada teve o seu valor máximo e indica esse valor, dando como resultado o par (data, valor):
def maxChuva(tabMeteo):
    if not tabMeteo:
        return (None, None)

    (data_max, tmin, tmax, prec_max) = tabMeteo[0]
    for (data, tmin, tmax, prec) in tabMeteo:
        if prec > prec_max:
            data_max = data
            prec_max = prec

    return (data_max, prec_max)
print(maxChuva(tabMeteo1))

#1.g) Define uma função que recebe uma tabela meteorológica e um limite `p` e retorna uma lista de pares [(data, precipitação)] correspondente aos dias em que a precipitação foi superior a `p`.
def diasChuvosos(tabMeteo, p):
    res = []
    for (data, tmin, tmax, prec) in tabMeteo:
        if prec > p:
            res.append((data, prec))
    return res

p = 0.05
print(diasChuvosos(tabMeteo1, p))

# 1.h) Maior número consecutivo de dias com precipitação < p
def maxPeriodoCalor(tabMeteo, p):
    max_run = 0
    cur = 0

    for (data, tmin, tmax, prec) in tabMeteo:
        if prec < p:        # se este dia tem precipitação abaixo do limite
            cur += 1        # aumenta a sequência atual
            if cur > max_run:
                max_run = cur
        else:
            cur = 0         #se choveu mais que p, quebra a sequência

    return max_run
print(maxPeriodoCalor(tabMeteo1, 0.05))

# 1.i) Gráficos das temperaturas e precipitação
def grafTabMeteo(tabMeteo):
    if plt is None:
        print("Erro: matplotlib não está instalado. Não é possível gerar gráficos.")
        return

    datas = []
    tmins = []
    tmaxs = []
    precs = []

    for (data, tmin, tmax, prec) in tabMeteo:
        datas.append(f"{data[0]}-{data[1]:02d}-{data[2]:02d}")
        tmins.append(tmin)
        tmaxs.append(tmax)
        precs.append(prec)

    # Gráfico da temperatura mínima
    plt.figure()
    plt.plot(datas, tmins, marker='o')
    plt.title("Temperatura Mínima")
    plt.xlabel("Data")
    plt.ylabel("Temp Mínima (°C)")
    plt.xticks(rotation=45)
    plt.tight_layout()
    plt.show()

    # Gráfico da temperatura máxima
    plt.figure()
    plt.plot(datas, tmaxs, marker='o')
    plt.title("Temperatura Máxima")
    plt.xlabel("Data")
    plt.ylabel("Temp Máxima (°C)")
    plt.xticks(rotation=45)
    plt.tight_layout()
    plt.show()

    # Gráfico da precipitação
    plt.figure()
    plt.plot(datas, precs, marker='o')
    plt.title("Precipitação")
    plt.xlabel("Data")
    plt.ylabel("Precipitação")
    plt.xticks(rotation=45)
    plt.tight_layout()
    plt.show()

# 1.j) Aplicação
def main():
    tabela = tabMeteo1.copy()

    while True:
        print("\n--- Menu Meteorologia ---")
        print("1 - Médias")
        print("2 - Guardar num ficheiro JSON")
        print("3 - Carregar num ficheiro JSON")
        print("4 - Temperatura mínima (absoluta)")
        print("5 - Amplitude térmica")
        print("6 - Valor máximo da precipitação")
        print("7 - Dias chuvosos (>p)")
        print("8 - Máximo período de tempo sem chuva (<p)")
        print("9 - Visualizar gráficos")
        print("0 - Sair")

        op = input("Opção: ")

        if op == "0":
            break
        elif op == "1":
            print(medias(tabela))
        elif op == "2":
            guardaTabMeteo(tabela, "meteorologia.json")
            print("Guardado.")
        elif op == "3":
            tabela = carregaTabMeteo("meteorologia.json")
            print("Carregado.")
        elif op == "4":
            print(minMin(tabela))
        elif op == "5":
            print(amplTerm(tabela))
        elif op == "6":
            print(maxChuva(tabela))
        elif op == "7":
            p = float(input("Limite p: "))
            print(diasChuvosos(tabela, p))
        elif op == "8":
            p = float(input("Limite p: "))
            print(maxPeriodoCalor(tabela, p))
        elif op == "9":
            grafTabMeteo(tabela)

if __name__ == "__main__":
    main()
