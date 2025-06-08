🌱 Atualmente, estou aprendendo desenvolvimento web full-stack React
💞️ Procuro colaborar em projetos reais
📫 Como entrar em contato comigo:Emersonsouza1168@gmail.com ou entre em contato comigo no nmero 61998330539
😄 Pronomes: Ele
⚡ Curiosidade aprendendo o básico!

tshared/schema";

export interface IStorage {
  // Clients
  getClients(): Promise<Client[]>;
  getClient(id: number): Promise<Client | undefined>;
  createClient(client: InsertClient): Promise<Client>;
  updateClient(id: number, client: Partial<InsertClient>): Promise<Client | undefined>;
  deleteClient(id: number): Promise<boolean>;

  // Properties
  getProperties(): Promise<Property[]>;
  getProperty(id: number): Promise<Property | undefined>;
  createProperty(property: InsertProperty): Promise<Property>;
  updateProperty(id: number, property: Partial<InsertProperty>): Promise<Property | undefined>;
  deleteProperty(id: number): Promise<boolean>;

  // Deals
  getDeals(): Promise<Deal[]>;
  getDealsWithDetails(): Promise<DealWithDetails[]>;
  getDeal(id: number): Promise<Deal | undefined>;
  getDealWithDetails(id: number): Promise<DealWithDetails | undefined>;
  createDeal(deal: InsertDeal): Promise<Deal>;
  updateDeal(id: number, deal: Partial<InsertDeal>): Promise<Deal | undefined>;
  deleteDeal(id: number): Promise<boolean>;

  // Audit Logs
  getAuditLogs(): Promise<AuditLog[]>;
  createAuditLog(log: InsertAuditLog): Promise<AuditLog>;
}

export class MemStorage implements IStorage {
  private clients: Map<number, Client>;
  private properties: Map<number, Property>;
  private deals: Map<number, Deal>;
  private auditLogs: Map<number, AuditLog>;
  private currentClientId: number;
  private currentPropertyId: number;
  private currentDealId: number;
  private currentAuditLogId: number;

  constructor() {
    this.clients = new Map();
    this.properties = new Map();
    this.deals = new Map();
    this.auditLogs = new Map();
    this.currentClientId = 1;
    this.currentPropertyId = 1;
    this.currentDealId = 1;
    this.currentAuditLogId = 1;
  }

  // Clients
  async getClients(): Promise<Client[]> {
    return Array.from(this.clients.values());
  }

  async getClient(id: number): Promise<Client | undefined> {
    return this.clients.get(id);
  }

  async createClient(insertClient: InsertClient): Promise<Client> {
    const id = this.currentClientId++;
    const client: Client = {
      ...insertClient,
      id,
      createdAt: new Date(),
    };
    this.clients.set(id, client);
    return client;
  }

  async updateClient(id: number, updateData: Partial<InsertClient>): Promise<Client | undefined> {
    const client = this.clients.get(id);
    if (!client) return undefined;

    const updatedClient = { ...client, ...updateData };
    this.clients.set(id, updatedClient);
    return updatedClient;
  }

  async deleteClient(id: number): Promise<boolean> {
    return this.clients.delete(id);
  }

  // Properties
  async getProperties(): Promise<Property[]> {
    return Array.from(this.properties.values());
  }

  async getProperty(id: number): Promise<Property | undefined> {
    return this.properties.get(id);
  }

  async createProperty(insertProperty: InsertProperty): Promise<Property> {
    const id = this.currentPropertyId++;
    const property: Property = {
      ...insertProperty,
      id,
      createdAt: new Date(),
    };
    this.properties.set(id, property);
    return property;
  }

  async updateProperty(id: number, updateData: Partial<InsertProperty>): Promise<Property | undefined> {
    const property = this.properties.get(id);
    if (!property) return undefined;

    const updatedProperty = { ...property, ...updateData };
    this.properties.set(id, updatedProperty);
    return updatedProperty;
  }

  async deleteProperty(id: number): Promise<boolean> {
    return this.properties.delete(id);
  }

  // Deals
  async getDeals(): Promise<Deal[]> {
    return Array.from(this.deals.values());
  }

  async getDealsWithDetails(): Promise<DealWithDetails[]> {
    const dealsArray = Array.from(this.deals.values());
    const dealsWithDetails: DealWithDetails[] = [];

    for (const deal of dealsArray) {
      const client = this.clients.get(deal.clientId);
      const property = this.properties.get(deal.propertyId);
      
      if (client && property) {
        dealsWithDetails.push({
          ...deal,
          client,
          property,
        });
      }
    }

    return dealsWithDetails;
  }

  async getDeal(id: number): Promise<Deal | undefined> {
    return this.deals.get(id);
  }

  async getDealWithDetails(id: number): Promise<DealWithDetails | undefined> {
    const deal = this.deals.get(id);
    if (!deal) return undefined;

    const client = this.clients.get(deal.clientId);
    const property = this.properties.get(deal.propertyId);

    if (!client || !property) return undefined;

    return {
      ...deal,
      client,
      property,
    };
  }

  async createDeal(insertDeal: InsertDeal): Promise<Deal> {
    const id = this.currentDealId++;
    const now = new Date();
    const deal: Deal = {
      ...insertDeal,
      id,
      createdAt: now,
      updatedAt: now,
    };
    this.deals.set(id, deal);
    return deal;
  }

  async updateDeal(id: number, updateData: Partial<InsertDeal>): Promise<Deal | undefined> {
    const deal = this.deals.get(id);
    if (!deal) return undefined;

    const updatedDeal = { 
      ...deal, 
      ...updateData, 
      updatedAt: new Date() 
    };
    this.deals.set(id, updatedDeal);
    return updatedDeal;
  }

  async deleteDeal(id: number): Promise<boolean> {
    return this.deals.delete(id);
  }

  // Audit Logs
  async getAuditLogs(): Promise<AuditLog[]> {
    return Array.from(this.auditLogs.values()).sort((a, b) => 
      b.timestamp.getTime() - a.timestamp.getTime()
    );
  }

  async createAuditLog(insertAuditLog: InsertAuditLog): Promise<AuditLog> {
    const id = this.currentAuditLogId++;
    const auditLog: AuditLog = {
      ...insertAuditLog,
      id,
      timestamp: new Date(),
    };
    this.auditLogs.set(id, auditLog);
    return auditLog;
  }
}

export const storage = new MemStorage();
