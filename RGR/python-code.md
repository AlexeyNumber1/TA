import random
import string
import sys
import time
import tracemalloc
import matplotlib.pyplot as plt


def naive_search(text, pattern):
    n = len(text)
    m = len(pattern)
    if m == 0 or m > n:
        return [], 0

    matches = []
    comparisons = 0

    for i in range(n - m + 1):
        j = 0
        while j < m:
            comparisons += 1
            if text[i + j] == pattern[j]:
                j += 1
            else:
                break
        if j == m:
            matches.append(i)

    return matches, comparisons


def build_bad_char_table(pattern):
    m = len(pattern)
    bad_char = {}
    for j in range(m - 1):
        bad_char[pattern[j]] = m - 1 - j
    return bad_char


def boyer_moore_search(text, pattern):
    n = len(text)
    m = len(pattern)
    if m == 0 or m > n:
        return [], 0, 0

    tracemalloc.start()
    bad_char = build_bad_char_table(pattern)
    _, peak_mem = tracemalloc.get_traced_memory()
    tracemalloc.stop()

    dict_mem = sys.getsizeof(bad_char) + sum(
        sys.getsizeof(k) + sys.getsizeof(v) for k, v in bad_char.items()
    )
    extra_memory = max(dict_mem, peak_mem)

    matches = []
    comparisons = 0
    i = 0

    while i <= n - m:
        j = m - 1
        while j >= 0:
            comparisons += 1
            if pattern[j] == text[i + j]:
                j -= 1
            else:
                break
        if j < 0:
            matches.append(i)
            i += 1
        else:
            char_shift = bad_char.get(text[i + j], m)
            shift = max(1, char_shift - (m - 1 - j))
            i += shift

    return matches, comparisons, extra_memory


def generate_text(length, alphabet=string.ascii_lowercase):
    return "".join(random.choice(alphabet) for _ in range(length))


def measure_average_time(search_function, text, pattern, runs=5):
    total_time = 0.0
    last_result = None
    for _ in range(runs):
        start_time = time.perf_counter()
        last_result = search_function(text, pattern)
        end_time = time.perf_counter()
        total_time += end_time - start_time
    return total_time / runs, last_result


def parse_sizes(raw_input, default_list):
    cleaned = [int(x.strip()) for x in raw_input.split(",") if x.strip()]
    return cleaned if cleaned else default_list


def compare_search_algorithms():
    default_text_sizes = [1000, 5000, 10000, 30000, 50000]
    default_pattern_sizes = [5, 15, 30, 50, 100]

    text_sizes_str = input(
        "Введіть довжини текстів через кому (Enter для 1000,5000,10000,30000,50000): "
    )
    text_sizes = parse_sizes(text_sizes_str, default_text_sizes)

    pattern_sizes_str = input(
        "Введіть довжини підрядків через кому (Enter для 5,15,30,50,100): "
    )
    pattern_sizes = parse_sizes(pattern_sizes_str, default_pattern_sizes)

    fixed_m = 15
    naive_times_n = []
    bm_times_n = []

    print("\n--- Залежність часу від довжини тексту n (m = 15) ---")
    for n in text_sizes:
        text = generate_text(n)
        pattern = text[n // 2 : n // 2 + fixed_m] if n >= fixed_m else text

        t_naive, (m_naive, c_naive) = measure_average_time(naive_search, text, pattern)
        t_bm, (m_bm, c_bm, mem_bm) = measure_average_time(boyer_moore_search, text, pattern)

        naive_times_n.append(t_naive)
        bm_times_n.append(t_bm)

        print(f"n = {n} | Збіги однакові: {m_naive == m_bm} ({len(m_naive)} шт.)")
        print(f"  Naive: {t_naive:.8f} с | порівнянь: {c_naive}")
        print(f"  BM:    {t_bm:.8f} с | порівнянь: {c_bm} | пам'ять таблиці: {mem_bm / 1024:.4f} KB")

    fixed_n = 50000
    base_text = generate_text(fixed_n)
    naive_times_m = []
    bm_times_m = []

    print("\n--- Залежність часу від довжини підрядка m (n = 50000) ---")
    for m in pattern_sizes:
        pattern = base_text[1000 : 1000 + m]

        t_naive, (_, c_naive) = measure_average_time(naive_search, base_text, pattern)
        t_bm, (_, c_bm, mem_bm) = measure_average_time(boyer_moore_search, base_text, pattern)

        naive_times_m.append(t_naive)
        bm_times_m.append(t_bm)

        print(f"m = {m}")
        print(f"  Naive: {t_naive:.8f} с | порівнянь: {c_naive}")
        print(f"  BM:    {t_bm:.8f} с | порівнянь: {c_bm} | пам'ять таблиці: {mem_bm / 1024:.4f} KB")

    plt.figure(figsize=(10, 5))
    plt.plot(text_sizes, naive_times_n, marker="o", label="Naive Search")
    plt.plot(text_sizes, bm_times_n, marker="o", label="Boyer-Moore Search")
    plt.xlabel("Довжина тексту (n)")
    plt.ylabel("Час виконання (с)")
    plt.title("Залежність часу пошуку від довжини тексту (m = 15)")
    plt.grid(True)
    plt.legend()
    plt.show()

    plt.figure(figsize=(10, 5))
    plt.plot(pattern_sizes, naive_times_m, marker="o", label="Naive Search")
    plt.plot(pattern_sizes, bm_times_m, marker="o", label="Boyer-Moore Search")
    plt.xlabel("Довжина підрядка (m)")
    plt.ylabel("Час виконання (с)")
    plt.title("Залежність часу пошуку від довжини підрядка (n = 50000)")
    plt.grid(True)
    plt.legend()
    plt.show()


if __name__ == "__main__":
    compare_search_algorithms()
