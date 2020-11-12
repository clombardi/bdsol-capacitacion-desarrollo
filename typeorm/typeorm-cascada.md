---
layout: default
---

# Cascada
Consideremos estos dos tipos de entidad.
```typescript
@Entity({ name: 'agencies' })
export class Agency {
    @PrimaryGeneratedColumn()
    id: number

    // ... otros atributos
    @OneToMany(() => OfficeParty, party => party.agency, { cascade: true })
    parties: OfficeParty[]
}

```
