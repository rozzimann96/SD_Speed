This is my first try for native MacOS (arm64) app disk / memory card speed benchmarking. Maybe someone find it useful since there is no another free alternative.

Becasuse I don't have App certificate, app is signed ad-hoc, so for running it's need to right-click on app and from menu choose Open.
Or after moving app to your local "Applications" folder, you can use attached sign.sh script.

**App can do this: **

- **4 tests:** sequential write and read, random write and read in 4 KiB blocks (IOPS + latency).
- **Two profiles:**
  - **Card / USB**: sequential 4 MiB blocks with QD1, random 4K with QD1.
  - **SSD / NVMe**: sequential 8 MiB blocks with QD4, random 4K with QD32.

  The profile is selected automatically based on the drive type, and each profile remembers its own test file size (default 1 GiB for cards, 4 GiB for SSDs).
- **Live display:** gauges with automatic scaling, a speed-versus-time graph (showing, for example, a slowdown after the SSD cache fills up), progress, and an estimate of remaining time.
- **SD Card Class Estimation:** C10, U1/U3, V6–V90, A1/A2. This is an approximate estimate based on measured values.
- **Comparison** with typical devices (USB 2.0, SD UHS-I/II, HDD, SATA SSD, NVMe) on a logarithmic axis.
- **Device detection:** type (card / USB / external / internal / network), file system, protocol (Secure Digital, USB…), SSD/HDD, model, and BSD name. A newly inserted card is automatically selected.
- **Test history** with export to CSV (separator `;`, decimal point, suitable for Czech Excel) and copying results as text.
- **Eject the card** with a single click after the test, so you can insert the next one right away.
- **Test any folder**, e.g., on a network volume.
- **Command-line mode** for scripts (see below).
The test is **non-destructive**. It writes only to a temporary hidden file `.SDSpeed-xxxx.tmp`, which is deleted after the test, upon interruption, or in the event of an error. Any remnants left after a potential application crash are cleaned up during the next test.


**Working principle:**

The goal is to obtain accurate figures that reflect the actual device, not the system cache:

- The test file is opened with `F_NOCACHE` (bypasses the macOS unified cache) and with read-ahead disabled (`F_RDAHEAD`). Before each read, any cached pages of the file are invalidated.
- Writes are completed with a call to `F_FULLFSYNC`, which also flushes the disk’s own cache. This time is **included** in the result.
- Additionally, random 4K writes are opened with `O_DSYNC`. On Apple Silicon, the kernel uses 16KiB pages, and file systems such as exFAT/FAT (which run via FSKit in macOS 26+) would otherwise silently buffer small writes. With `O_DSYNC`, each write to the card is completed before the next one begins, so the value corresponds to the actual QD1 latency.
- Each sequential pass reads or writes the file **exactly once**. Repeated reading and writing of the same small file would inflate the results due to the SSD or HDD controller cache.
- Random tests run for a set duration (5–6 s) at random, aligned positions throughout the file. The queue depth (QD) is emulated by a corresponding number of threads.
- The data is random and uncompressible; each block is unique.
- Speeds are in **MB/s = 10⁶ B/s** (as in CrystalDiskMark or Blackmagic), file sizes in MiB/GiB.

Tips: Use at least 1 GiB for memory cards, and 4 GiB or more for SSDs. For more stable results, set 3 passes (both the average and maximum are displayed).

<img width="1600" height="1042" alt="screenshot" src="https://github.com/user-attachments/assets/303767f5-d8fa-4960-a8ff-ee82544ddd30" />
