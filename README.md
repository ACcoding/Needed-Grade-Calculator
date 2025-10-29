import tkinter as tk
import math  # for ceiling function

root = tk.Tk()
root.title("GPA Calculator")
root.geometry("550x700")

# --- Page ---
page = tk.Frame(root, bg="lightblue", width=1000, height=1000)
page.grid(row=0, column=0, sticky="nsew")

# --- Table ---
tableFrame = tk.Frame(page, bg="white", bd=2, relief="solid")
tableFrame.place(x=150, y=150)

colNames = ["Grade", "Weight"]
entries = []          # 2D list of Entry widgets
entryPositions = {}  # dict with "r,c" keys
MAX_ROWS = 5

# column headers
for c, name in enumerate(colNames):
    colLabel = tk.Label(tableFrame, text=name, font=("Arial", 10, "bold"), bg="white")
    colLabel.grid(row=0, column=c+1, padx=5, pady=5)

# initial 2 rows
baseRowNames = ["Quarter 1", "Quarter 2"]
for r, name in enumerate(baseRowNames):
    rowLabel = tk.Label(tableFrame, text=name, font=("Arial", 10, "bold"), bg="white")
    rowLabel.grid(row=r+1, column=0, padx=5, pady=5)
    rowEntries = []
    for c in range(len(colNames)):
        e = tk.Entry(tableFrame, width=10)
        e.grid(row=r+1, column=c+1, padx=5, pady=5)
        e.insert(0, "0")
        rowEntries.append(e)
        entryPositions[f"{r+1},{c+1}"] = e
    entries.append(rowEntries)

resultLabel = tk.Label(page, text="Grade Needed:", bg="lightblue", font=("Arial", 12))
calcButton = tk.Button(page, text="Calculate")
findFinal = tk.Button(page, text="Find needed Final grade.")
findMidterm = tk.Button(page, text="Find needed Midterm grade.")

label2Info = tk.Label(page, text="Enter the prompted grades and their weights.", bg="lightblue", font=("Arial", 12))
label2Info.place(x=110, y=120)

def repositionControls():
    y_offset = 200 + (len(entries) + 1) * 30
    calcButton.place(x=235, y=y_offset-20)
    resultLabel.place(x=215, y=y_offset + 60)
    updateButtons()

def updateButtons():
    y_offset = 200 + (len(entries) + 1) * 30
    if len(entries) == 2:
        findFinal.place(x=200, y=y_offset+15)
        findMidterm.place_forget()
    elif len(entries) >= MAX_ROWS:
        findFinal.place_forget()
        findMidterm.place(x=195, y=y_offset+30)

# --- Add/Remove Row ---
def addRow():
    for i in range(3):
        r = len(entries)
        if r >= MAX_ROWS:
            return
        if r == 2:
            name = "Midterm"
        elif r > 2:
            name = f"Quarter {r}"
        else:
            name = f"Quarter {r+1}"

        rowLabel = tk.Label(tableFrame, text=name, font=("Arial", 10, "bold"), bg="white")
        rowLabel.grid(row=r+1, column=0, padx=5, pady=5)

        rowEntries = []
        for c in range(len(colNames)):
            e = tk.Entry(tableFrame, width=10)
            e.grid(row=r+1, column=c+1, padx=5, pady=5)
            e.insert(0, "0")
            rowEntries.append(e)
            entryPositions[f"{r+1},{c+1}"] = e
        entries.append(rowEntries)
        repositionControls()

def removeRow():
    for i in range(3):
        if len(entries) > 2:
            r = len(entries)
            rowEntries = entries.pop()
            for widget in rowEntries:
                widget.destroy()
            for widget in tableFrame.grid_slaves(row=r, column=0):
                widget.destroy()
            repositionControls()

findFinal.config(command=addRow)
findMidterm.config(command=removeRow)

def calculateNeededGrade():
    try:
        numRows = len(entries)
        targets = [90, 80, 70, 60, 59]
        neededGrades = {}

        if numRows == 2:
            totalSoFar = 0
            weightSum = 0
            for r in range(2):
                val = float(entries[r][0].get())
                weight = float(entries[r][1].get())
                if weight > 1:
                    weight /= 100
                totalSoFar += val * weight
                weightSum += weight
            remaining_weight = 1 - weightSum
            for t in targets:
                neededGrades[t] = (t - totalSoFar) / remaining_weight
            label_text = "Midterm Grade Needed:"
        else:
            total_so_far = 0
            totalWeight = 0
            for r in range(min(numRows, 5)):
                val = float(entries[r][0].get())
                weight = float(entries[r][1].get())
                if weight >= 10:
                    weight /= 10
                if numRows <= 5:
                    totalWeight += weight
                    total_so_far += val * weight
                if numRows == 5:
                    totalWeight += (weight / 2)
                    total_so_far += val * (weight / 2)
            for t in targets:
                neededGrades[t] = (t - total_so_far)
                if neededGrades[t] > 0:
                    neededGrades[t] = neededGrades[t] / (1 - totalWeight)
                else:
                    neededGrades[t] = 0
            label_text = "Final Grade Needed:"

        # Build results (rounded up to nearest whole number)
        results = []
        for t in targets:
            val = neededGrades[t]
            val = math.ceil(val)  # round up to nearest whole number
            if val <= 0:
                results.append(f"You cannot get below a {t}.")
                break
            elif val > 100:
                if t <= 59:
                    results.append("You will fail no matter what.")
                else:
                    results.append(f"{t}: Not achievable")
                break
            else:
                results.append(f"{t}: You need {val}%.")
        resultLabel.config(text=label_text + "\n" + "\n".join(results))

    except Exception as e:
        resultLabel.config(text=f"Error: {e}")
    resultLabel.place(x=180, y=(260 + (len(entries) + 1) * 30))

calcButton.config(command=calculateNeededGrade)

repositionControls()

root.mainloop()
