# sorting-algorrithm-analyser
  # its a sorting algortuihm performance analyser using bubble  sort, insertion sort, merge sort , quuick sort  etc to sort an array and cocurrenty shows  the time  
  # taken along  with gui support  which shows  a chart  representation  using python.
import time
import random
import sys
import argparse
from copy import deepcopy
import matplotlib.pyplot as plt
import tkinter as tk
from tkinter import ttk, messagebox, scrolledtext, simpledialog
from matplotlib.backends.backend_tkagg import FigureCanvasTkAgg

# ======================
# Sorting Algorithms
# ======================

def bubble_sort(arr):
    n = len(arr)
    for i in range(n):
        swapped = False
        for j in range(n-i-1):
            if arr[j] > arr[j+1]:
                arr[j], arr[j+1] = arr[j+1], arr[j]
                swapped = True
        if not swapped:
            break
    return arr

def selection_sort(arr):
    n = len(arr)
    for i in range(n):
        min_idx = i
        for j in range(i+1, n):
            if arr[j] < arr[min_idx]:
                min_idx = j
        arr[i], arr[min_idx] = arr[min_idx], arr[i]
    return arr

def insertion_sort(arr):
    for i in range(1, len(arr)):
        key = arr[i]
        j = i-1
        while j >= 0 and key < arr[j]:
            arr[j+1] = arr[j]
            j -= 1
        arr[j+1] = key
    return arr

def merge_sort(arr):
    if len(arr) > 1:
        mid = len(arr)//2
        L = arr[:mid]
        R = arr[mid:]

        merge_sort(L)
        merge_sort(R)

        i = j = k = 0
        while i < len(L) and j < len(R):
            if L[i] < R[j]:
                arr[k] = L[i]
                i += 1
            else:
                arr[k] = R[j]
                j += 1
            k += 1

        while i < len(L):
            arr[k] = L[i]
            i += 1
            k += 1

        while j < len(R):
            arr[k] = R[j]
            j += 1
            k += 1
    return arr

def quick_sort(arr):
    if len(arr) <= 1:
        return arr
    pivot = arr[len(arr)//2]
    left = [x for x in arr if x < pivot]
    middle = [x for x in arr if x == pivot]
    right = [x for x in arr if x > pivot]
    return quick_sort(left) + middle + quick_sort(right)

def python_timsort(arr):
    arr.sort()
    return arr

# ======================
# Core Functionality
# ======================

class SortingAnalyzer:
    ALGORITHMS = {
        "Bubble Sort": bubble_sort,
        "Selection Sort": selection_sort,
        "Insertion Sort": insertion_sort,
        "Merge Sort": merge_sort,
        "Quick Sort": quick_sort,
        "Python Timsort": python_timsort
    }
    
    DATA_TYPES = ["random", "sorted", "reverse", "nearly_sorted", "few_unique"]
    
    @staticmethod
    def generate_data(size, data_type='random'):
        if data_type == 'random':
            return [random.randint(0, size*10) for _ in range(size)]
        elif data_type == 'sorted':
            return [i for i in range(size)]
        elif data_type == 'reverse':
            return [i for i in range(size, 0, -1)]
        elif data_type == 'nearly_sorted':
            arr = list(range(size))
            for _ in range(max(1, size//10)):
                i, j = random.randint(0, size-1), random.randint(0, size-1)
                arr[i], arr[j] = arr[j], arr[i]
            return arr
        elif data_type == 'few_unique':
            unique_values = [random.randint(1, 5) for _ in range(5)]
            return [random.choice(unique_values) for _ in range(size)]
        else:
            raise ValueError(f"Unknown data type: {data_type}")
    
    @classmethod
    def sort_array(cls, algorithm_name, arr):
        if algorithm_name not in cls.ALGORITHMS:
            raise ValueError(f"Unknown algorithm: {algorithm_name}")
        
        start_time = time.time()
        sorted_arr = cls.ALGORITHMS[algorithm_name](arr.copy())
        time_taken = time.time() - start_time
        
        return sorted_arr, time_taken
    
    @classmethod
    def analyze_performance(cls, sizes, data_type='random', selected_algs=None):
        if selected_algs is None:
            selected_algs = cls.ALGORITHMS.keys()
            
        results = {name: [] for name in selected_algs}
        text_results = []
        
        for size in sizes:
            data = cls.generate_data(size, data_type)
            text_results.append(f"\nInput Size: {size} ({data_type} data)")
            
            for name in selected_algs:
                func = cls.ALGORITHMS[name]
                arr = deepcopy(data)
                start_time = time.time()
                
                if name == "Quick Sort":
                    arr = func(arr)
                else:
                    func(arr)
                    
                time_taken = time.time() - start_time
                results[name].append(time_taken)
                text_results.append(f"{name}: {time_taken:.6f} seconds")
        
        return results, "\n".join(text_results)

# ======================
# CLI Interface
# ======================

def display_results_table(results):
    print("\nPerformance Results:")
    headers = ["Algorithm"] + [f"Size {size}" for size in results["sizes"]]
    print("\t".join(headers))
    
    for algo in results["algorithms"]:
        row = [algo] + [f"{t:.6f}" for t in results[algo]]
        print("\t".join(row))

def plot_performance(results):
    plt.figure(figsize=(10, 6))
    for algo in results["algorithms"]:
        plt.plot(results["sizes"], results[algo], marker='o', label=algo)
    
    plt.title(f'Sorting Algorithm Performance ({results["data_type"].capitalize()} Data)')
    plt.xlabel('Input Size')
    plt.ylabel('Execution Time (seconds)')
    plt.legend()
    plt.grid(True)
    plt.show()

def cli_sort_array():
    print("\n=== Array Sorting ===")
    input_str = input("Enter array elements (comma separated): ").strip()
    try:
        arr = [int(x.strip()) if x.strip().isdigit() else float(x.strip()) for x in input_str.split(',')]
    except ValueError:
        print("Error: Please enter only numbers separated by commas")
        return
    
    print("\nAvailable algorithms:")
    for i, algo in enumerate(SortingAnalyzer.ALGORITHMS.keys(), 1):
        print(f"{i}. {algo}")
    
    try:
        choice = int(input("Select algorithm (1-6): ").strip())
        algo_name = list(SortingAnalyzer.ALGORITHMS.keys())[choice-1]
    except (ValueError, IndexError):
        print("Invalid selection")
        return
    
    sorted_arr, time_taken = SortingAnalyzer.sort_array(algo_name, arr)
    
    print("\nOriginal array:", arr)
    print("Sorted array:", sorted_arr)
    print(f"Time taken: {time_taken:.6f} seconds")
    print("Correctly sorted:", sorted_arr == sorted(arr.copy()))

def cli_analyze_performance():
    print("\n=== Performance Analysis ===")
    sizes_input = input("Enter sizes to test (comma separated, e.g., 100,500,1000): ").strip()
    try:
        sizes = [int(size.strip()) for size in sizes_input.split(',')]
    except ValueError:
        print("Error: Please enter only integers separated by commas")
        return
    
    print("\nAvailable data types:", ", ".join(SortingAnalyzer.DATA_TYPES))
    data_type = input("Select data type: ").strip().lower()
    if data_type not in SortingAnalyzer.DATA_TYPES:
        print("Invalid data type, using 'random'")
        data_type = 'random'
    
    print("\nAvailable algorithms:")
    for i, algo in enumerate(SortingAnalyzer.ALGORITHMS.keys(), 1):
        print(f"{i}. {algo}")
    
    selected = input("Select algorithms (comma separated, or 'all'): ").strip()
    if selected.lower() == 'all':
        selected_algs = list(SortingAnalyzer.ALGORITHMS.keys())
    else:
        try:
            selected_indices = [int(x.strip())-1 for x in selected.split(',')]
            selected_algs = [list(SortingAnalyzer.ALGORITHMS.keys())[i] for i in selected_indices]
        except (ValueError, IndexError):
            print("Invalid selection, using all algorithms")
            selected_algs = list(SortingAnalyzer.ALGORITHMS.keys())
    
    results, text_results = SortingAnalyzer.analyze_performance(sizes, data_type, selected_algs)
    
    print("\n=== Results ===")
    print(text_results)
    
    # Prepare data for plotting
    plot_data = {
        "sizes": sizes,
        "data_type": data_type,
        "algorithms": selected_algs
    }
    for algo in selected_algs:
        plot_data[algo] = results[algo]
    
    # Display table
    display_results_table(plot_data)
    
    # Show plot
    try:
        plot_performance(plot_data)
    except ImportError:
        print("\nNote: Install matplotlib to see graphical results")

def run_cli():
    while True:
        print("\n=== Sorting Algorithm Analyzer ===")
        print("1. Sort an array")
        print("2. Analyze algorithm performance")
        print("3. Exit")
        
        choice = input("Enter your choice (1-3): ").strip()
        
        if choice == '1':
            cli_sort_array()
        elif choice == '2':
            cli_analyze_performance()
        elif choice == '3':
            print("Exiting...")
            break
        else:
            print("Invalid choice, please try again")

# ======================
# GUI Interface
# ======================

class SortingAnalyzerApp:
    def __init__(self, root):
        self.root = root
        self.root.title("Sorting Algorithm Analyzer")
        self.root.geometry("1000x800")
        
        self.notebook = ttk.Notebook(root)
        self.notebook.pack(fill='both', expand=True)
        
        # Create tabs
        self.create_sorting_tab()
        self.create_analysis_tab()
        
        # Status bar
        self.status_var = tk.StringVar()
        self.status_bar = ttk.Label(root, textvariable=self.status_var, relief='sunken')
        self.status_bar.pack(fill='x')
    
    def create_sorting_tab(self):
        tab = ttk.Frame(self.notebook)
        self.notebook.add(tab, text='Array Sorting')
        
        # Input array
        ttk.Label(tab, text="Input Array:").grid(row=0, column=0, padx=5, pady=5, sticky='e')
        self.array_entry = ttk.Entry(tab, width=40)
        self.array_entry.grid(row=0, column=1, padx=5, pady=5, sticky='w')
        self.array_entry.insert(0, "3, 1, 4, 1, 5, 9, 2, 6, 5, 3")
        
        # Algorithm selection
        ttk.Label(tab, text="Algorithm:").grid(row=1, column=0, padx=5, pady=5, sticky='e')
        self.algorithm = ttk.Combobox(tab, values=list(SortingAnalyzer.ALGORITHMS.keys()), state='readonly')
        self.algorithm.grid(row=1, column=1, padx=5, pady=5, sticky='w')
        self.algorithm.current(0)
        
        # Sort button
        ttk.Button(tab, text="Sort Array", command=self.run_sorting).grid(row=2, column=0, columnspan=2, pady=10)
        
        # Results
        self.sorting_results = scrolledtext.ScrolledText(tab, height=10, wrap='word')
        self.sorting_results.grid(row=3, column=0, columnspan=2, padx=5, pady=5, sticky='nsew')
        
        tab.grid_rowconfigure(3, weight=1)
        tab.grid_columnconfigure(1, weight=1)
    
    def create_analysis_tab(self):
        tab = ttk.Frame(self.notebook)
        self.notebook.add(tab, text='Performance Analysis')
        
        # Input sizes
        ttk.Label(tab, text="Input Sizes:").grid(row=0, column=0, padx=5, pady=5, sticky='e')
        self.sizes_entry = ttk.Entry(tab)
        self.sizes_entry.grid(row=0, column=1, padx=5, pady=5, sticky='w')
        self.sizes_entry.insert(0, "100, 500, 1000, 2000, 5000")
        
        # Data type
        ttk.Label(tab, text="Data Type:").grid(row=1, column=0, padx=5, pady=5, sticky='e')
        self.data_type = ttk.Combobox(tab, values=SortingAnalyzer.DATA_TYPES, state='readonly')
        self.data_type.grid(row=1, column=1, padx=5, pady=5, sticky='w')
        self.data_type.current(0)
        
        # Algorithm selection
        ttk.Label(tab, text="Algorithms:").grid(row=2, column=0, padx=5, pady=5, sticky='ne')
        self.algorithm_vars = {}
        algos_frame = ttk.Frame(tab)
        algos_frame.grid(row=2, column=1, padx=5, pady=5, sticky='w')
        
        for i, algo in enumerate(SortingAnalyzer.ALGORITHMS.keys()):
            self.algorithm_vars[algo] = tk.BooleanVar(value=True)
            cb = ttk.Checkbutton(algos_frame, text=algo, variable=self.algorithm_vars[algo])
            cb.grid(row=i//3, column=i%3, sticky='w', padx=5)
        
        # Analyze button
        ttk.Button(tab, text="Analyze Performance", command=self.run_analysis).grid(row=3, column=0, columnspan=2, pady=10)
        
        # Results
        self.analysis_results = scrolledtext.ScrolledText(tab, height=10, wrap='word')
        self.analysis_results.grid(row=4, column=0, columnspan=2, padx=5, pady=5, sticky='nsew')
        
        # Plot frame
        self.figure = plt.Figure(figsize=(8, 4), dpi=100)
        self.ax = self.figure.add_subplot(111)
        self.canvas = FigureCanvasTkAgg(self.figure, tab)
        self.canvas.get_tk_widget().grid(row=5, column=0, columnspan=2, padx=5, pady=5, sticky='nsew')
        
        tab.grid_rowconfigure(4, weight=1)
        tab.grid_rowconfigure(5, weight=1)
        tab.grid_columnconfigure(1, weight=1)
    
    def run_sorting(self):
        try:
            input_str = self.array_entry.get().strip()
            arr = [int(x.strip()) if x.strip().isdigit() else float(x.strip()) for x in input_str.split(',')]
            algo_name = self.algorithm.get()
            
            self.status_var.set("Sorting array...")
            self.root.update()
            
            sorted_arr, time_taken = SortingAnalyzer.sort_array(algo_name, arr)
            
            self.sorting_results.delete(1.0, tk.END)
            self.sorting_results.insert(tk.END, "=== Sorting Results ===\n")
            self.sorting_results.insert(tk.END, f"Algorithm: {algo_name}\n")
            self.sorting_results.insert(tk.END, f"Original array: {arr}\n")
            self.sorting_results.insert(tk.END, f"Sorted array: {sorted_arr}\n")
            self.sorting_results.insert(tk.END, f"Time taken: {time_taken:.6f} seconds\n")
            self.sorting_results.insert(tk.END, f"Correctly sorted: {sorted_arr == sorted(arr.copy())}")
            
            self.status_var.set("Sorting completed")
            
        except ValueError as e:
            messagebox.showerror("Error", f"Invalid input: {str(e)}")
            self.status_var.set("Error in sorting")
        except Exception as e:
            messagebox.showerror("Error", f"An error occurred: {str(e)}")
            self.status_var.set("Error in sorting")
    
    def run_analysis(self):
        try:
            sizes = [int(size.strip()) for size in self.sizes_entry.get().split(',')]
            data_type = self.data_type.get()
            selected_algs = [algo for algo, var in self.algorithm_vars.items() if var.get()]
            
            if not selected_algs:
                messagebox.showwarning("Warning", "Please select at least one algorithm")
                return
            
            self.status_var.set("Running analysis...")
            self.root.update()
            
            results, text_results = SortingAnalyzer.analyze_performance(sizes, data_type, selected_algs)
            
            self.analysis_results.delete(1.0, tk.END)
            self.analysis_results.insert(tk.END, text_results)
            
            # Update plot
            self.ax.clear()
            for algo in selected_algs:
                self.ax.plot(sizes, results[algo], marker='o', label=algo)
            
            self.ax.set_title(f'Sorting Algorithm Performance ({data_type.capitalize()} Data)')
            self.ax.set_xlabel('Input Size')
            self.ax.set_ylabel('Execution Time (seconds)')
            self.ax.legend()
            self.ax.grid(True)
            self.canvas.draw()
            
            self.status_var.set("Analysis completed")
            
        except ValueError as e:
            messagebox.showerror("Error", f"Invalid input: {str(e)}")
            self.status_var.set("Error in analysis")
        except Exception as e:
            messagebox.showerror("Error", f"An error occurred: {str(e)}")
            self.status_var.set("Error in analysis")

def run_gui():
    root = tk.Tk()
    app = SortingAnalyzerApp(root)
    root.mainloop()

# ======================
# Main Execution
# ======================

if __name__ == "__main__":
    if len(sys.argv) > 1 and sys.argv[1] == "--gui":
        run_gui()
    else:
        run_cli()
