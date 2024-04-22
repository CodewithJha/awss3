# awss3
import tkinter as tk
from tkinter import messagebox, filedialog
import boto3

class S3BrowserApp:
    def __init__(self, master):
        self.master = master
        self.master.title("AWS S3 Browser")

        # Initialize AWS S3 client
        try:
            self.s3 = boto3.client('s3', region_name='ap-south-1')
        except Exception as e:
            messagebox.showerror("Error", f"Failed to initialize AWS S3 client: {str(e)}")
            return

        # Create GUI elements
        self.bucket_listbox = tk.Listbox(master, width=50)
        self.bucket_listbox.pack(pady=10)

        self.refresh_button = tk.Button(master, text="Refresh Buckets", command=self.refresh_buckets)
        self.refresh_button.pack()

        self.list_button = tk.Button(master, text="List Bucket Contents", command=self.list_bucket_contents)
        self.list_button.pack()

        self.upload_button = tk.Button(master, text="Upload File", command=self.upload_file)
        self.upload_button.pack()

        self.delete_button = tk.Button(master, text="Delete Item", command=self.delete_item)
        self.delete_button.pack()

    def refresh_buckets(self):
        self.bucket_listbox.delete(0, tk.END)
        try:
            buckets = self.s3.list_buckets()['Buckets']
            for bucket in buckets:
                self.bucket_listbox.insert(tk.END, bucket['Name'])
        except Exception as e:
            messagebox.showerror("Error", f"Failed to retrieve buckets: {str(e)}")

    def list_bucket_contents(self):
        selected_bucket = self.get_selected_bucket()
        if selected_bucket:
            try:
                contents = self.s3.list_objects_v2(Bucket=selected_bucket)
                if 'Contents' in contents:
                    for item in contents['Contents']:
                        print(item['Key'])
                        # Here you can display or process the contents as needed
                else:
                    messagebox.showinfo("Info", "Bucket is empty")
            except Exception as e:
                messagebox.showerror("Error", f"Failed to list bucket contents: {str(e)}")

    def upload_file(self):
        selected_bucket = self.get_selected_bucket()
        if selected_bucket:
            try:
                file_path = filedialog.askopenfilename()
                if file_path:
                    file_name = file_path.split('/')[-1]
                    with open(file_path, 'rb') as file:
                        self.s3.upload_fileobj(file, selected_bucket, file_name)
                    messagebox.showinfo("Info", "File uploaded successfully")
            except Exception as e:
                messagebox.showerror("Error", f"Failed to upload file: {str(e)}")

    def delete_item(self):
        selected_bucket = self.get_selected_bucket()
        if selected_bucket:
            selected_item = self.bucket_listbox.get(tk.ACTIVE)
            if selected_item:
                if messagebox.askyesno("Confirmation", f"Are you sure you want to delete {selected_item}?"):
                    try:
                        self.s3.delete_object(Bucket=selected_bucket, Key=selected_item)
                        messagebox.showinfo("Info", "Item deleted successfully")
                        self.refresh_buckets()
                    except Exception as e:
                        messagebox.showerror("Error", f"Failed to delete item: {str(e)}")

    def get_selected_bucket(self):
        selected_bucket_index = self.bucket_listbox.curselection()
        if selected_bucket_index:
            return self.bucket_listbox.get(selected_bucket_index)
        else:
            messagebox.showerror("Error", "Please select a bucket")
            return None

def main():
    root = tk.Tk()
    app = S3BrowserApp(root)
    root.mainloop()

if __name__ == "__main__":
    main()
