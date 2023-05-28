---
layout: post
title:  "OpenBabel最小环主要逻辑代码"
comments: true
categories: Others
---

主要在mol和ring
```cpp
#include <openbabel/mol.h>
#include <openbabel/generic.h>
#include <openbabel/atom.h>
#include <openbabel/bond.h>
#include <openbabel/ring.h>
#include <openbabel/obiter.h>
```

```cpp
//self write
vector<OBRing*> vr = mol.GetSSSR();
```
# 1. GetSSSR
```cpp
vector<OBRing*>& OBMol::GetSSSR()
{
	if (!HasSSSRPerceived())
	FindSSSR();//				<----

	OBRingData* rd = nullptr;
	if (!HasData("SSSR")) {
		rd = new OBRingData();
		rd->SetAttribute("SSSR");
		SetData(rd);
	}

	rd = (OBRingData*)GetData("SSSR");
	rd->SetOrigin(perceived);
	return(rd->GetData());
}
```
# 2. FindSSSR
```cpp
void OBMol::FindSSSR()
{
	if (HasSSSRPerceived())
		return;
	SetSSSRPerceived();
	obErrorLog.ThrowError(__FUNCTION__,
		"Ran OpenBabel::FindSSSR", obAuditMsg);

	// Delete any old data before we start finding new rings
	// The following procedure is slow
	// So if client code is multi-threaded, we don't want to make them wait
	if (HasData("SSSR")) {
		DeleteData("SSSR");
	}

	OBRing* ring;
	vector<OBRing*>::iterator j;

	//get Fr�rejacque taking int account multiple possible spanning graphs
	int frj = DetermineFRJ(*this);//				<----
	if (frj)
	{
		vector<OBRing*> vr;
		FindRingAtomsAndBonds();

		OBBond* bond;
		vector<OBBond*> cbonds;
		vector<OBBond*>::iterator k;

		//restrict search for rings around closure bonds
		for (bond = BeginBond(k); bond; bond = NextBond(k))
			if (bond->IsClosure())
				cbonds.push_back(bond);

		if (!cbonds.empty())
		{
			OBRingSearch rs;
			//search for all rings about closures
			vector<OBBond*>::iterator i;

			for (i = cbonds.begin(); i != cbonds.end(); ++i)
				rs.AddRingFromClosure(*this, (OBBond*)*i);

			rs.SortRings();
			rs.RemoveRedundant(frj);
			//store the SSSR set

			for (j = rs.BeginRings(); j != rs.EndRings(); ++j)
			{
				ring = new OBRing((*j)->_path, NumAtoms() + 1);
				ring->SetParent(this);
				vr.push_back(ring);
			}
			//rs.WriteRings();
		}

		OBRingData* rd = new OBRingData();
		rd->SetOrigin(perceived); // to separate from user or file input
		rd->SetAttribute("SSSR");
		rd->SetData(vr);
		SetData(rd);
	}
}

```
# 3. DetermineFRJ
```cpp
static int DetermineFRJ(OBMol& mol)
{
	if (!mol.HasClosureBondsPerceived())
		return (int)FindRingAtomsAndBonds2(mol);//				<----

	int frj = 0;
	OBBond* bond;
	vector<OBBond*>::iterator j;
	for (bond = mol.BeginBond(j); bond; bond = mol.NextBond(j))
		if (bond->IsClosure()) // bond->HasFlag(OB_CLOSURE_BOND)?
			frj++;
	return frj;
}
```
```cpp
static int FindRings(OBAtom* atom, int* avisit, unsigned char* bvisit, unsigned int& frj, int depth)
{
	OBBond* bond;
	int result = -1;
	vector<OBBond*>::iterator k;
	for (bond = atom->BeginBond(k); bond; bond = atom->NextBond(k)) {
		unsigned int bidx = bond->GetIdx();
		if (bvisit[bidx] == 0) {
			bvisit[bidx] = 1;
			OBAtom* nbor = bond->GetNbrAtom(atom);
			unsigned int nidx = nbor->GetIdx();
			int nvisit = avisit[nidx];
			if (nvisit == 0) {
				avisit[nidx] = depth + 1;
				nvisit = FindRings(nbor, avisit, bvisit, frj, depth + 1);
				if (nvisit > 0) {
					if (nvisit <= depth) {
						bond->SetInRing();
						if (result < 0 || nvisit < result)
							result = nvisit;
					}
				}
			}
			else {
				if (result < 0 || nvisit < result)
					result = nvisit;
				bond->SetClosure();
				bond->SetInRing();
				frj++;
			}
		}
	}
	if (result > 0 && result <= depth)
		atom->SetInRing();
	return result;
}
```
# 4. FindRingAtomsAndBonds2
```cpp
static unsigned int FindRingAtomsAndBonds2(OBMol& mol)
{
	mol.SetRingAtomsAndBondsPerceived(); // mol.SetFlag(OB_RINGFLAGS_MOL);
	mol.SetClosureBondsPerceived();      // mol.SetFlag(OB_CLOSURE_MOL);

	FOR_ATOMS_OF_MOL(atom, mol)
		atom->SetInRing(false);
	FOR_BONDS_OF_MOL(bond, mol) {
		bond->SetInRing(false);
		bond->SetClosure(false);
	}

	unsigned int bsize = mol.NumBonds() + 1;
	unsigned char* bvisit = (unsigned char*)malloc(bsize);
	memset(bvisit, 0, bsize);

	unsigned int acount = mol.NumAtoms();
	unsigned int asize = (unsigned int)((acount + 1) * sizeof(int));
	int* avisit = (int*)malloc(asize);
	memset(avisit, 0, asize);

	unsigned int frj = 0;
	for (unsigned int i = 1; i <= acount; i++)
		if (avisit[i] == 0) {
			avisit[i] = 1;
			OBAtom* atom = mol.GetAtom(i);
			FindRings(atom, avisit, bvisit, frj, 1);//				<----
		}
	free(avisit);
	free(bvisit);
	return frj;
}
```
# 5. FindRings
```cpp
static int FindRings(OBAtom* atom, int* avisit, unsigned char* bvisit,
	unsigned int& frj, int depth)
{
	OBBond* bond;
	int result = -1;
	vector<OBBond*>::iterator k;
	for (bond = atom->BeginBond(k); bond; bond = atom->NextBond(k)) {
		unsigned int bidx = bond->GetIdx();
		if (bvisit[bidx] == 0) {
			bvisit[bidx] = 1;
			OBAtom* nbor = bond->GetNbrAtom(atom);
			unsigned int nidx = nbor->GetIdx();
			int nvisit = avisit[nidx];
			if (nvisit == 0) {
				avisit[nidx] = depth + 1;
				nvisit = FindRings(nbor, avisit, bvisit, frj, depth + 1);// <----
				if (nvisit > 0) {
					if (nvisit <= depth) {
						bond->SetInRing();
						if (result < 0 || nvisit < result)
							result = nvisit;
					}
				}
			}
			else {
				if (result < 0 || nvisit < result)
					result = nvisit;
				bond->SetClosure();
				bond->SetInRing();
				frj++;
			}
		}
	}
	if (result > 0 && result <= depth)
		atom->SetInRing();
	return result;
}
```