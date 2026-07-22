from pathlib import Path
from dataclasses import dataclass


@dataclass
class ScanResult:
    scanned: int = 0
    empty: int = 0
    removed: int = 0


class EmptyFolderCleaner:

    def __init__(self, root):
        self.root = Path(root)
        self.result = ScanResult()
        self.empty_folders = []

    def scan(self):
        self.empty_folders.clearr()

        # از پایین به بالا اسکن می‌کنیم
        for folder in sorted(
            self.root.rglob("*"),
            key=lambda p: len(p.parts),
            reverse=True
        ):

            if not folder.is_dir():
                continue

            self.result.scanned += 1

            try:
                if not any(folder.iterdir()):
                    self.empty_folders.append(folder)
                    self.result.empty += 1
            except PermissionError:
                pass

    def report(self):
        print("\n========== REPORT ==========")
        print(f"Scanned folders : {self.result.scanned}")
        print(f"Empty folders   : {self.result.empty}")
        print("=" * 28)

        for folder in self.empty_folders:
            print(folder)

    def remove(self):
        for folder in self.empty_folders:
            try:
                folder.rmdir()
                self.result.removed += 1
            except OSError:
                pass

        print(f"\nRemoved {self.result.removed} folders.")


def main():

    path = input("Directory: ").strip()

    cleaner = EmptyFolderCleaner(path)

    cleaner.scan()

    cleaner.report()

    if cleaner.empty_folders:

        answer = input(
            "\nDelete all empty folders? (y/n): "
        ).lower()

        if answer == "y":
            cleaner.remove()


if __name__ == "__main__":
    main()
