# Livewire-API

## Livewire
```bash
<?php

namespace App\Livewire; // Menetapkan namespace untuk file ini.

use App\Models\UnitKerja as UnitKerjaModel; // Menggunakan model UnitKerja dan memberi alias.
use Livewire\Component; // Menggunakan komponen Livewire.
use Illuminate\Http\Request; // Menggunakan class Request untuk menangani permintaan HTTP.
use Illuminate\Validation\ValidationException; // Untuk menangani validasi yang gagal.

class UnitKerja extends Component // Deklarasi class utama UnitKerja yang merupakan komponen Livewire.
{
    public $nama_unit_kerja = ''; // Properti publik untuk menyimpan nama unit kerja.
    public $unit_kerjas; // Menyimpan daftar unit kerja dari database.
    public $edit_id = null; // ID dari data yang sedang diedit.
    public $is_edit_mode = false; // Menandakan apakah sedang dalam mode edit.

    protected $rules = [ // Aturan validasi data input.
        'nama_unit_kerja' => 'required|string|max:255',
    ];

    protected $messages = [ // Pesan kustom untuk validasi.
        'nama_unit_kerja.required' => 'Nama unit kerja wajib diisi.',
        'nama_unit_kerja.string' => 'Nama unit kerja harus berupa teks.',
        'nama_unit_kerja.max' => 'Nama unit kerja tidak boleh lebih dari 255 karakter.',
    ];

    public function mount() // Method yang dijalankan saat komponen diinisialisasi.
    {
        $this->loadData(); // Memuat data unit kerja.
    }

    public function loadData() // Memuat data unit kerja yang belum dihapus.
    {
        $this->unit_kerjas = UnitKerjaModel::where('deleted_at', null)->get();
    }

    public function store() // Menyimpan data baru.
    {
        try {
            $this->validate(); // Validasi data input.

            UnitKerjaModel::create([
                'nama_unit_kerja' => $this->nama_unit_kerja,
                'created_at' => now(), // Timestamp sekarang.
            ]);

            session()->flash('message', 'Data berhasil disimpan'); // Pesan sukses.
            $this->resetForm(); // Reset form input.
            $this->loadData(); // Refresh data.
        } catch (ValidationException $e) {
            $this->addError('nama_unit_kerja', $e->errors()['nama_unit_kerja'][0]);
        } catch (\Throwable $th) {
            session()->flash('error', 'Terjadi kesalahan: ' . $th->getMessage());
        }
    }

    // API: Simpan data via request HTTP.
    public function apiStore(Request $request)
    {
        try {
            $validated = $this->validateRequest($request, [
                'nama_unit_kerja' => 'required|string|max:255',
            ]);

            $unit_kerja = UnitKerjaModel::create([
                'nama_unit_kerja' => $validated['nama_unit_kerja'],
                'created_at' => now(),
            ]);

            return response()->json([
                'status' => 200,
                'message' => 'Data berhasil disimpan',
                'errors' => [],
                'data' => $unit_kerja,
            ]);
        } catch (ValidationException $e) {
            return response()->json([
                'status' => 422,
                'message' => $e->getMessage(),
                'errors' => $e->errors(),
            ]);
        } catch (\Throwable $th) {
            return response()->json([
                'status' => 500,
                'message' => $th->getMessage(),
                'errors' => [],
                'data' => [],
            ]);
        }
    }

    public function apiGetAll() // API: Ambil semua data unit kerja.
    {
        try {
            $data = UnitKerjaModel::where('deleted_at', null)->get();
            return response()->json([
                'status' => 200,
                'message' => 'Data berhasil diambil',
                'errors' => [],
                'data' => $data,
            ]);
        } catch (\Throwable $th) {
            return response()->json([
                'status' => 500,
                'message' => $th->getMessage(),
                'errors' => [],
                'data' => [],
            ]);
        }
    }

    public function apiEdit($id) // API: Ambil data berdasarkan ID.
    {
        try {
            $data = UnitKerjaModel::findOrFail($id);
            return response()->json([
                'status' => 200,
                'message' => 'Data berhasil diambil',
                'errors' => [],
                'data' => $data,
            ]);
        } catch (\Throwable $th) {
            return response()->json([
                'status' => 500,
                'message' => $th->getMessage(),
                'errors' => [],
                'data' => [],
            ]);
        }
    }

    public function apiUpdate(Request $request, $id) // API: Update data berdasarkan ID.
    {
        try {
            $validated = $this->validateRequest($request, [
                'nama_unit_kerja' => 'required|string|max:255',
            ]);

            UnitKerjaModel::where('id', $id)->update([
                'nama_unit_kerja' => $validated['nama_unit_kerja'],
                'updated_at' => now(),
            ]);

            return response()->json([
                'status' => 200,
                'message' => 'Data berhasil diupdate',
                'errors' => [],
                'data' => $request->all(),
            ]);
        } catch (ValidationException $e) {
            return response()->json([
                'status' => 422,
                'message' => $e->getMessage(),
                'errors' => $e->errors(),
            ]);
        } catch (\Throwable $th) {
            return response()->json([
                'status' => 500,
                'message' => $th->getMessage(),
                'errors' => [],
                'data' => [],
            ]);
        }
    }

    public function apiDelete($id) // API: Hapus (soft delete) data berdasarkan ID.
    {
        try {
            UnitKerjaModel::where('id', $id)->update(['deleted_at' => now()]);
            return response()->json([
                'status' => 200,
                'message' => 'Data berhasil dihapus',
                'errors' => [],
                'data' => [],
            ]);
        } catch (\Throwable $th) {
            return response()->json([
                'status' => 500,
                'message' => $th->getMessage(),
                'errors' => [],
                'data' => [],
            ]);
        }
    }

    public function edit($id) // Ambil data untuk form edit.
    {
        try {
            $unit_kerja = UnitKerjaModel::findOrFail($id);
            $this->nama_unit_kerja = $unit_kerja->nama_unit_kerja;
            $this->edit_id = $id;
            $this->is_edit_mode = true;
        } catch (\Throwable $th) {
            session()->flash('error', 'Gagal mengambil data: ' . $th->getMessage());
        }
    }

    public function update() // Simpan hasil edit.
    {
        try {
            $this->validate();

            UnitKerjaModel::where('id', $this->edit_id)->update([
                'nama_unit_kerja' => $this->nama_unit_kerja,
                'updated_at' => now(),
            ]);

            session()->flash('message', 'Data berhasil diupdate');
            $this->resetForm();
            $this->loadData();
        } catch (ValidationException $e) {
            $this->addError('nama_unit_kerja', $e->errors()['nama_unit_kerja'][0]);
        } catch (\Throwable $th) {
            session()->flash('error', 'Terjadi kesalahan: ' . $th->getMessage());
        }
    }

    public function delete($id) // Soft delete data dari UI.
    {
        try {
            UnitKerjaModel::where('id', $id)->update(['deleted_at' => now()]);
            session()->flash('message', 'Data berhasil dihapus');
            $this->loadData();
        } catch (\Throwable $th) {
            session()->flash('error', 'Gagal menghapus data: ' . $th->getMessage());
        }
    }

    public function resetForm() // Mengatur ulang nilai input dan error.
    {
        $this->nama_unit_kerja = '';
        $this->edit_id = null;
        $this->is_edit_mode = false;
        $this->resetErrorBag(); // Reset error validasi.
    }

    // Validasi khusus untuk API
    protected function validateRequest(Request $request, array $rules)
    {
        $validator = \Validator::make($request->all(), $rules, $this->messages);
        if ($validator->fails()) {
            throw new ValidationException($validator);
        }
        return $validator->validated(); // Mengembalikan data yang telah tervalidasi.
    }

    public function render() // Render tampilan komponen Livewire.
    {
        return view('livewire.unit-kerja')->extends('layout.layouts');
    }
}


```

## blade
```bash
<div>
    <div class="container mt-5">
        <h2 class="mb-4">Manajemen Unit Kerja</h2>

        <!-- Menampilkan pesan sukses -->
        @if (session('message'))
            <div class="alert alert-success alert-dismissible fade show" role="alert">
                {{ session('message') }} <!-- Menampilkan pesan sukses dari session -->
                <button type="button" class="btn-close" data-bs-dismiss="alert" aria-label="Close"></button>
            </div>
        @endif

        <!-- Menampilkan pesan error -->
        @if (session('error'))
            <div class="alert alert-danger alert-dismissible fade show" role="alert">
                {{ session('error') }} <!-- Menampilkan pesan error dari session -->
                <button type="button" class="btn-close" data-bs-dismiss="alert" aria-label="Close"></button>
            </div>
        @endif

        <!-- Form untuk tambah atau edit data unit kerja -->
        <div class="card mb-4">
            <div class="card-header">{{ $is_edit_mode ? 'Edit Unit Kerja' : 'Tambah Unit Kerja' }}</div> <!-- Judul form menyesuaikan mode -->
            <div class="card-body">
                <form wire:submit.prevent="{{ $is_edit_mode ? 'update' : 'store' }}"> <!-- Kirim form ke method Livewire update atau store -->
                    <div class="mb-3">
                        <label for="nama_unit_kerja" class="form-label">Nama Unit Kerja</label>
                        <input type="text" class="form-control @error('nama_unit_kerja') is-invalid @enderror" 
                               id="nama_unit_kerja" wire:model="nama_unit_kerja"> <!-- Input terhubung dengan property Livewire -->
                        @error('nama_unit_kerja')
                            <div class="invalid-feedback">{{ $message }}</div> <!-- Validasi error -->
                        @enderror
                    </div>
                    <button type="submit" class="btn btn-primary">{{ $is_edit_mode ? 'Update' : 'Simpan' }}</button> <!-- Tombol simpan/update -->
                    @if ($is_edit_mode)
                        <button type="button" class="btn btn-secondary" wire:click="resetForm">Batal</button> <!-- Tombol batal jika dalam edit mode -->
                    @endif
                </form>
            </div>
        </div>

        <!-- Tabel daftar unit kerja -->
        <div class="card">
            <div class="card-header">Daftar Unit Kerja</div>
            <div class="card-body">
                <table class="table table-bordered table-striped">
                    <thead>
                        <tr>
                            <th>No</th>
                            <th>Nama Unit Kerja</th>
                            <th>Aksi</th> <!-- Kolom aksi untuk edit/hapus -->
                        </tr>
                    </thead>
                    <tbody>
                        @forelse ($unit_kerjas as $index => $unit_kerja) <!-- Looping data unit kerja -->
                            <tr>
                                <td>{{ $index + 1 }}</td> <!-- Nomor urut -->
                                <td>{{ $unit_kerja->nama_unit_kerja }}</td> <!-- Nama unit kerja -->
                                <td>
                                    <button class="btn btn-sm btn-warning" wire:click="edit({{ $unit_kerja->id }})">Edit</button> <!-- Panggil method edit -->
                                    <button class="btn btn-sm btn-danger" wire:click="delete({{ $unit_kerja->id }})" 
                                            onclick="return confirm('Apakah Anda yakin ingin menghapus data ini?')">Hapus</button> <!-- Konfirmasi hapus dan panggil method delete -->
                                </td>
                            </tr>
                        @empty
                            <tr>
                                <td colspan="3" class="text-center">Tidak ada data.</td> <!-- Tampilkan pesan jika data kosong -->
                            </tr>
                        @endforelse
                    </tbody>
                </table>
            </div>
        </div>
    </div>
</div>


```
## web.php
```bash
Route::get('/unit-kerja', UnitKerja::class)->name('unit-kerja');
```

## api.php
```bash
use App\Livewire\UnitKerja;



Route::prefix('unit_kerja')->group(function () {
    Route::post('/store', [UnitKerja::class, 'apiStore'])->name('api.unit_kerja.store');
    Route::get('/getAll', [UnitKerja::class, 'apiGetAll'])->name('api.unit_kerja.getAll');
    Route::get('/edit/{id}', [UnitKerja::class, 'apiEdit'])->name('api.unit_kerja.edit');
    Route::put('/update/{id}', [UnitKerja::class, 'apiUpdate'])->name('api.unit_kerja.update');
    Route::delete('/delete/{id}', [UnitKerja::class, 'apiDelete'])->name('api.unit_kerja.delete');
});
```

## Testing API
```bash
### Get All Unit Kerja
GET http://localhost:8000/api/unit_kerja/getAll
Accept: application/json

###

### Store Unit Kerja
POST http://localhost:8000/api/unit_kerja/store
Content-Type: application/json
Accept: application/json

{
  "nama_unit_kerja": "Teknologi Informasi"
}

###

### Edit Unit Kerja (Ganti :id sesuai data yang ada)
GET http://localhost:8000/api/unit_kerja/edit/1
Accept: application/json

###

### Update Unit Kerja (Ganti :id sesuai data yang ada)
PUT http://localhost:8000/api/unit_kerja/update/1
Content-Type: application/json
Accept: application/json

{
  "nama_unit_kerja": "Teknologi Informasi & Komunikasi"
}

###

### Delete Unit Kerja (Ganti :id sesuai data yang ada)
DELETE http://localhost:8000/api/unit_kerja/delete/1
Accept: application/json
```
![image](https://github.com/user-attachments/assets/afac76f4-5da0-4112-be06-44f56c49c508)

