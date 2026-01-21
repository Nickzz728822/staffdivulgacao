import React, { useState } from 'react';
import { useAdmin } from '@/contexts/AdminContext';
import { Button } from '@/components/ui/button';
import { Tabs, TabsContent, TabsList, TabsTrigger } from '@/components/ui/tabs';
import { Trash2, Save, X, Edit2, Plus, Upload, Image as ImageIcon } from 'lucide-react';
import { useToast } from '@/components/ui/use-toast';

const AdminPanel = () => {
  const { 
    templates, deleteTemplate, updateTemplate, addTemplate,
    effects, deleteEffect, updateEffect, addEffect,
    presets, deletePreset, updatePreset, addPreset,
    uploadImage
  } = useAdmin();
  
  const { toast } = useToast();

  const EditableTable = ({ items, onDelete, onUpdate }) => {
    const [editingId, setEditingId] = useState(null);
    const [editForm, setEditForm] = useState({});
    const [uploading, setUploading] = useState(false);

    const startEdit = (item) => {
      setEditingId(item.id);
      setEditForm(item);
    };

    const cancelEdit = () => {
      setEditingId(null);
      setEditForm({});
      setUploading(false);
    };

    const saveEdit = () => {
      onUpdate(editingId, editForm);
      setEditingId(null);
      toast({
        title: "Item atualizado",
        description: "As alterações foram salvas com sucesso."
      });
    };

    const handleImageUpload = async (e) => {
      const file = e.target.files[0];
      if (!file) return;

      setUploading(true);
      try {
        const base64 = await uploadImage(file);
        setEditForm(prev => ({ ...prev, imageUrl: base64 }));
        toast({
          title: "Imagem carregada",
          description: "Imagem adicionada com sucesso."
        });
      } catch (error) {
        toast({
          title: "Erro no upload",
          description: error.message,
          variant: "destructive"
        });
      } finally {
        setUploading(false);
      }
    };

    return (
      <div className="bg-white/5 rounded-xl border border-white/10 overflow-hidden">
        <table className="w-full text-left">
          <thead className="bg-white/5 text-gray-400 text-sm">
            <tr>
              <th className="p-4 w-24">Imagem</th>
              <th className="p-4">Nome</th>
              <th className="p-4">Descrição</th>
              <th className="p-4 text-right">Ações</th>
            </tr>
          </thead>
          <tbody className="divide-y divide-white/10">
            {items.map((item) => (
              <tr key={item.id} className="hover:bg-white/5 transition-colors">
                <td className="p-4 align-top">
                  {editingId === item.id ? (
                    <div className="space-y-2">
                      <div className="w-16 h-16 rounded bg-black/40 flex items-center justify-center overflow-hidden border border-white/10">
                        {editForm.imageUrl ? (
                          <img src={editForm.imageUrl} alt="Preview" className="w-full h-full object-cover" />
                        ) : (
                          <ImageIcon className="w-6 h-6 text-gray-500" />
                        )}
                      </div>
                      <div className="relative">
                        <input
                          type="file"
                          accept="image/*"
                          onChange={handleImageUpload}
                          className="hidden"
                          id={`file-upload-${item.id}`}
                          disabled={uploading}
                        />
                        <label
                          htmlFor={`file-upload-${item.id}`}
                          className="block w-full text-center px-2 py-1 bg-purple-500/20 hover:bg-purple-500/30 text-purple-400 text-xs rounded cursor-pointer transition-colors"
                        >
                          {uploading ? '...' : 'Upload'}
                        </label>
                      </div>
                      {editForm.imageUrl && (
                        <button
                          onClick={() => setEditForm(prev => ({ ...prev, imageUrl: '' }))}
                          className="block w-full text-center text-[10px] text-red-400 hover:text-red-300"
                        >
                          Remover
                        </button>
                      )}
                    </div>
                  ) : (
                    <div className="w-16 h-16 rounded bg-black/20 flex items-center justify-center overflow-hidden border border-white/10">
                      {item.imageUrl ? (
                        <img src={item.imageUrl} alt={item.name} className="w-full h-full object-cover" />
                      ) : (
                        <ImageIcon className="w-6 h-6 text-gray-600" />
                      )}
                    </div>
                  )}
                </td>
                <td className="p-4 align-top">
                  {editingId === item.id ? (
                    <input 
                      className="bg-black/20 border border-purple-500/50 rounded px-2 py-1 text-white w-full"
                      value={editForm.name}
                      onChange={(e) => setEditForm({...editForm, name: e.target.value})}
                    />
                  ) : (
                    <span className="text-white font-medium block">{item.name}</span>
                  )}
                </td>
                <td className="p-4 align-top">
                  {editingId === item.id ? (
                    <textarea 
                      className="bg-black/20 border border-purple-500/50 rounded px-2 py-1 text-white w-full resize-y min-h-[80px]"
                      value={editForm.description || ''}
                      onChange={(e) => setEditForm({...editForm, description: e.target.value})}
                    />
                  ) : (
                    <span className="text-gray-400 text-sm line-clamp-3">{item.description || 'Sem descrição'}</span>
                  )}
                </td>
                <td className="p-4 text-right align-top">
                  <div className="flex justify-end gap-2">
                    {editingId === item.id ? (
                      <>
                        <Button size="sm" onClick={saveEdit} className="bg-green-600 hover:bg-green-700 h-8 w-8 p-0">
                          <Save className="w-4 h-4" />
                        </Button>
                        <Button size="sm" variant="ghost" onClick={cancelEdit} className="h-8 w-8 p-0">
                          <X className="w-4 h-4" />
                        </Button>
                      </>
                    ) : (
                      <>
                        <Button size="sm" variant="ghost" onClick={() => startEdit(item)} className="text-purple-400 hover:text-purple-300 h-8 w-8 p-0">
                          <Edit2 className="w-4 h-4" />
                        </Button>
                        <Button 
                          size="sm" 
                          variant="ghost" 
                          onClick={() => {
                            if(confirm('Tem certeza que deseja excluir este item?')) {
                              onDelete(item.id);
                              toast({ title: "Item excluído", variant: "destructive" });
                            }
                          }}
                          className="text-red-400 hover:text-red-300 h-8 w-8 p-0"
                        >
                          <Trash2 className="w-4 h-4" />
                        </Button>
                      </>
                    )}
                  </div>
                </td>
              </tr>
            ))}
          </tbody>
        </table>
      </div>
    );
  };

  const AddButton = ({ onClick, label }) => (
    <Button 
      onClick={onClick}
      className="bg-gradient-to-r from-purple-500 to-pink-600 hover:from-purple-600 hover:to-pink-700 mb-4"
    >
      <Plus className="w-4 h-4 mr-2" />
      {label}
    </Button>
  );

  return (
    <div className="space-y-6">
      <Tabs defaultValue="templates" className="w-full">
        <TabsList className="bg-white/5 border border-white/10 p-1">
          <TabsTrigger value="templates" className="data-[state=active]:bg-purple-600">Modelos</TabsTrigger>
          <TabsTrigger value="effects" className="data-[state=active]:bg-purple-600">Efeitos</TabsTrigger>
          <TabsTrigger value="presets" className="data-[state=active]:bg-purple-600">Presets</TabsTrigger>
        </TabsList>
        
        <TabsContent value="templates" className="mt-6">
          <AddButton 
            label="Adicionar Modelo" 
            onClick={() => {
              addTemplate({ name: 'Novo Modelo', description: 'Descrição aqui', category: 'Intro', imageUrl: '' });
              toast({ title: "Novo modelo criado", description: "Edite o item para adicionar detalhes e imagem." });
            }} 
          />
          <EditableTable items={templates} onDelete={deleteTemplate} onUpdate={updateTemplate} type="template" />
        </TabsContent>
        
        <TabsContent value="effects" className="mt-6">
          <AddButton 
            label="Adicionar Efeito" 
            onClick={() => {
              addEffect({ name: 'Novo Efeito', description: 'Descrição aqui', effect_type: 'Desfoque', imageUrl: '' });
              toast({ title: "Novo efeito criado", description: "Edite o item para adicionar detalhes e imagem." });
            }} 
          />
          <EditableTable items={effects} onDelete={deleteEffect} onUpdate={updateEffect} type="effect" />
        </TabsContent>

        <TabsContent value="presets" className="mt-6">
          <AddButton 
            label="Adicionar Preset" 
            onClick={() => {
              addPreset({ name: 'Novo Preset', brightness: 0, contrast: 0, saturation: 0, hue: 0, imageUrl: '' });
              toast({ title: "Novo preset criado", description: "Edite o item para adicionar detalhes e imagem." });
            }} 
          />
          <EditableTable items={presets} onDelete={deletePreset} onUpdate={updatePreset} type="preset" />
        </TabsContent>
      </Tabs>
    </div>
  );
};

export default AdminPanel;
