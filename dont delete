import os
import shutil
import tkinter as tk
from datetime import datetime
from tkinter import filedialog, messagebox, simpledialog, ttk


class FileManagerApp:
    def __init__(self, root):
        self.root = root
        self.root.title("File Manager")
        self.root.geometry("900x700")

        # Configuration
        self.target_folder = ""
        self.recently_added = []
        self.setup_ui()

        # Initialize with default target (e.g., Documents)
        self.initialize_target_folder()

    def setup_ui(self):
        # Create main frames
        control_frame = ttk.Frame(self.root, padding="10")
        control_frame.pack(fill=tk.X)

        result_frame = ttk.Frame(self.root)
        result_frame.pack(fill=tk.BOTH, expand=True, padx=10, pady=5)

        # Folder selection
        ttk.Label(control_frame, text="Target Location:").grid(
            row=0, column=0, sticky=tk.W
        )
        self.folder_path_var = tk.StringVar()
        ttk.Entry(control_frame, textvariable=self.folder_path_var, width=50).grid(
            row=0, column=1, padx=5
        )
        ttk.Button(
            control_frame, text="Change Location", command=self.change_target_folder
        ).grid(row=0, column=2)

        # Action buttons
        action_frame = ttk.Frame(control_frame)
        action_frame.grid(row=1, column=0, columnspan=3, pady=10)

        ttk.Button(action_frame, text="Add Files", command=self.add_files).pack(
            side=tk.LEFT, padx=5
        )
        ttk.Button(action_frame, text="Add Folder", command=self.add_folder).pack(
            side=tk.LEFT, padx=5
        )
        ttk.Button(
            action_frame, text="Create Subfolder", command=self.create_subfolder
        ).pack(side=tk.LEFT, padx=5)
        ttk.Button(action_frame, text="View Contents", command=self.view_contents).pack(
            side=tk.LEFT, padx=5
        )

        # Search functionality
        search_frame = ttk.Frame(control_frame)
        search_frame.grid(row=2, column=0, columnspan=3, pady=5)

        ttk.Label(search_frame, text="Search:").pack(side=tk.LEFT)
        self.search_entry = ttk.Entry(search_frame, width=40)
        self.search_entry.pack(side=tk.LEFT, padx=5)
        self.search_entry.bind("<Return>", lambda e: self.search_files())
        ttk.Button(search_frame, text="Search", command=self.search_files).pack(
            side=tk.LEFT
        )

        # Results display
        self.result_tree = ttk.Treeview(
            result_frame, columns=("Name", "Type", "Size", "Modified"), show="headings"
        )
        self.result_tree.heading("Name", text="Name")
        self.result_tree.heading("Type", text="Type")
        self.result_tree.heading("Size", text="Size")
        self.result_tree.heading("Modified", text="Modified")
        self.result_tree.column("Name", width=300)
        self.result_tree.column("Type", width=100)
        self.result_tree.column("Size", width=100)
        self.result_tree.column("Modified", width=150)

        scrollbar = ttk.Scrollbar(
            result_frame, orient="vertical", command=self.result_tree.yview
        )
        self.result_tree.configure(yscrollcommand=scrollbar.set)

        self.result_tree.pack(side=tk.LEFT, fill=tk.BOTH, expand=True)
        scrollbar.pack(side=tk.RIGHT, fill=tk.Y)

        # Context menu
        self.context_menu = tk.Menu(self.root, tearoff=0)
        self.context_menu.add_command(label="Open", command=self.open_selected)
        self.context_menu.add_command(label="Open Location", command=self.open_location)
        self.context_menu.add_command(label="Delete", command=self.delete_selected)

        self.result_tree.bind("<Button-3>", self.show_context_menu)
        self.result_tree.bind("<Double-1>", lambda e: self.open_selected())

    def initialize_target_folder(self):
        """Sets default target to user's Documents folder (or another default if preferred)."""
        default_path = os.path.join(os.path.expanduser("~"), "Documents")
        if os.path.exists(default_path):
            self.set_target_folder(default_path)
        else:
            self.change_target_folder()  # Let user choose if default doesn't exist

    def set_target_folder(self, path):
        """Updates the target folder and refreshes the view."""
        self.target_folder = path
        self.folder_path_var.set(path)
        self.view_contents()

    def change_target_folder(self):
        """Lets the user select a new target folder."""
        folder = filedialog.askdirectory(title="Select Target Folder")
        if folder:
            self.set_target_folder(folder)

    def add_files(self):
        """Adds selected files to the target folder."""
        if not self.target_folder:
            messagebox.showerror("Error", "No target folder selected")
            return

        files = filedialog.askopenfilenames(title="Select Files to Add")
        if not files:
            return

        success_count = 0
        for file_path in files:
            try:
                file_name = os.path.basename(file_path)
                dest_path = os.path.join(self.target_folder, file_name)

                # Handle duplicates
                counter = 1
                name, ext = os.path.splitext(file_name)
                while os.path.exists(dest_path):
                    new_name = f"{name}_{counter}{ext}"
                    dest_path = os.path.join(self.target_folder, new_name)
                    counter += 1

                shutil.copy2(file_path, dest_path)
                self.recently_added.append(
                    (file_name, datetime.now().strftime("%Y-%m-%d %H:%M:%S"))
                )
                success_count += 1
            except Exception as e:
                messagebox.showerror("Error", f"Failed to add {file_path}: {str(e)}")

        if success_count > 0:
            messagebox.showinfo("Success", f"Added {success_count} file(s)")
            self.view_contents()

    def add_folder(self):
        """Adds a folder to the target location."""
        if not self.target_folder:
            messagebox.showerror("Error", "No target folder selected")
            return

        folder_path = filedialog.askdirectory(title="Select Folder to Add")
        if not folder_path:
            return

        folder_name = os.path.basename(folder_path)
        dest_path = os.path.join(self.target_folder, folder_name)

        # Handle duplicates
        counter = 1
        original_dest = dest_path
        while os.path.exists(dest_path):
            dest_path = f"{original_dest}_{counter}"
            counter += 1

        try:
            shutil.copytree(folder_path, dest_path)
            self.recently_added.append(
                (folder_name + "/", datetime.now().strftime("%Y-%m-%d %H:%M:%S"))
            )
            messagebox.showinfo(
                "Success", f"Folder '{os.path.basename(dest_path)}' added successfully"
            )
            self.view_contents()
        except Exception as e:
            messagebox.showerror("Error", f"Failed to add folder: {str(e)}")

    def create_subfolder(self):
        """Creates a new subfolder in the target location."""
        if not self.target_folder:
            messagebox.showerror("Error", "No target folder selected")
            return

        folder_name = simpledialog.askstring(
            "Create Subfolder", "Enter subfolder name:"
        )
        if folder_name:
            try:
                new_folder = os.path.join(self.target_folder, folder_name)
                os.makedirs(new_folder, exist_ok=True)
                messagebox.showinfo("Success", f"Subfolder '{folder_name}' created")
                self.view_contents()
            except Exception as e:
                messagebox.showerror("Error", f"Failed to create subfolder: {str(e)}")

    def view_contents(self):
        """Displays contents of the target folder."""
        if not self.target_folder:
            return

        self.result_tree.delete(*self.result_tree.get_children())

        try:
            for item in os.listdir(self.target_folder):
                item_path = os.path.join(self.target_folder, item)
                item_type = "Folder" if os.path.isdir(item_path) else "File"

                if os.path.isdir(item_path):
                    size = ""
                else:
                    size = self.format_size(os.path.getsize(item_path))

                modified = datetime.fromtimestamp(os.path.getmtime(item_path)).strftime(
                    "%Y-%m-%d %H:%M:%S"
                )

                self.result_tree.insert(
                    "",
                    "end",
                    values=(item, item_type, size, modified),
                    tags=(item_type.lower(),),
                )
        except Exception as e:
            messagebox.showerror("Error", f"Failed to view contents: {str(e)}")

    def search_files(self):
        """Searches for files/folders matching the query."""
        query = self.search_entry.get().strip().lower()
        if not query:
            messagebox.showwarning("Warning", "Please enter a search term")
            return

        self.result_tree.delete(*self.result_tree.get_children())
        matches = []

        for root, dirs, files in os.walk(self.target_folder):
            for name in dirs + files:
                if query in name.lower():
                    item_path = os.path.join(root, name)
                    item_type = "Folder" if os.path.isdir(item_path) else "File"

                    if os.path.isdir(item_path):
                        size = ""
                    else:
                        size = self.format_size(os.path.getsize(item_path))

                    modified = datetime.fromtimestamp(
                        os.path.getmtime(item_path)
                    ).strftime("%Y-%m-%d %H:%M:%S")
                    relative_path = os.path.relpath(item_path, self.target_folder)

                    matches.append((relative_path, item_type, size, modified))

        if not matches:
            self.result_tree.insert("", "end", values=("No matches found", "", "", ""))
        else:
            for match in matches:
                self.result_tree.insert(
                    "", "end", values=match, tags=(match[1].lower(),)
                )

    def show_context_menu(self, event):
        """Shows right-click context menu."""
        item = self.result_tree.identify_row(event.y)
        if item:
            self.result_tree.selection_set(item)
            self.context_menu.post(event.x_root, event.y_root)

    def open_selected(self):
        """Opens the selected file/folder."""
        selected = self.result_tree.selection()
        if selected:
            item = self.result_tree.item(selected[0])
            name = item["values"][0]
            item_path = os.path.join(self.target_folder, name)

            try:
                if os.path.isdir(item_path):
                    self.set_target_folder(item_path)
                else:
                    os.startfile(item_path)
            except Exception as e:
                messagebox.showerror("Error", f"Failed to open: {str(e)}")

    def open_location(self):
        """Opens the parent folder of the selected item."""
        selected = self.result_tree.selection()
        if selected:
            item = self.result_tree.item(selected[0])
            name = item["values"][0]
            item_path = os.path.join(self.target_folder, name)

            try:
                if os.path.exists(item_path):
                    os.startfile(os.path.dirname(item_path))
            except Exception as e:
                messagebox.showerror("Error", f"Failed to open location: {str(e)}")

    def delete_selected(self):
        """Deletes the selected file/folder."""
        selected = self.result_tree.selection()
        if not selected:
            return

        item = self.result_tree.item(selected[0])
        name = item["values"][0]
        item_path = os.path.join(self.target_folder, name)

        if messagebox.askyesno(
            "Confirm Delete", f"Are you sure you want to delete '{name}'?"
        ):
            try:
                if os.path.isdir(item_path):
                    shutil.rmtree(item_path)
                else:
                    os.remove(item_path)
                self.view_contents()
                messagebox.showinfo("Success", f"'{name}' deleted successfully")
            except Exception as e:
                messagebox.showerror("Error", f"Failed to delete: {str(e)}")

    @staticmethod
    def format_size(size_bytes):
        """Convert file size to human-readable format."""
        for unit in ["bytes", "KB", "MB", "GB"]:
            if size_bytes < 1024.0:
                return f"{size_bytes:.1f} {unit}"
            size_bytes /= 1024.0
        return f"{size_bytes:.1f} TB"


if __name__ == "__main__":
    root = tk.Tk()
    app = FileManagerApp(root)
    root.mainloop()
