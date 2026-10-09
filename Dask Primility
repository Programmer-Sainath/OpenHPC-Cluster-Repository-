import random
import time
import os

from dask.distributed import Client, LocalCluster, as_completed


# Miller-Rabin primality test
def miller_rabin(n, k):

    if n <= 1:
        return False

    if n < 4:
        return True

    if n % 2 == 0:
        return False

    r = 0
    d = n - 1

    while d % 2 == 0:
        r += 1
        d //= 2

    for _ in range(k):

        a = random.randint(2, n - 2)

        x = pow(a, d, n)

        if x == 1 or x == n - 1:
            continue

        for _ in range(r - 1):
            x = pow(x, 2, n)

            if x == n - 1:
                break
        else:
            return False

    return True


BITSIZE = 1024
LOOPNUM = 20


def generate_prime_candidate():

    while True:

        p = random.getrandbits(BITSIZE)

        # force odd and force top bit
        p |= (1 << (BITSIZE - 1)) | 1

        start = time.perf_counter()
        miller_rabin(p, 20)
        elapsed = time.perf_counter() - start

        # keep candidates that take some work
        if elapsed > 0.01:
            return p


def findprime(loopnum):

    while True:
        a = generate_prime_candidate()
        if miller_rabin(a, loopnum):
            return a

        a -= 2


def main():

    workers = os.cpu_count()

    print(f"Starting Dask with {workers} workers")

    cluster = LocalCluster(
        n_workers=workers,
        threads_per_worker=1,
        processes=True
    )

    client = Client(cluster)

    print(client)

    futures = [
        client.submit(findprime, LOOPNUM) for x in range(workers)
    ]

    prime = None

    for future in as_completed(futures):

        prime = future.result()

        break

    for f in futures:
        f.cancel()

    client.close()
    cluster.close()
    return prime
