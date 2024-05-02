export interface User {
  id?: number;
  uuid?: string;
  firstName?: string;
  lastName?: string;
  matricule?: string;
  enabled?: boolean;
  email?: string;
  role?:string
  dateStatus?: Date;
  dateCreation?: Date;
  applications?: Application[];
}
export interface Application
{
  id: number;
  appName : string;
  logo: string;
  linkFront :  string;
  linkBack :  string;
  linkPilotage :  string;
  countTask : number;
  portalRequestDTOExternList : PortalRequestDto[];
}

export interface PortalRequestDto
{
  id: number,
  name : string;
  libelle: string,
  count: number,
  status: string,
  link: string
}

this.serviceUser.getUsers();

voila cette fonction me retourne liste de users 
je veux crée des charts avec highcharts-angular type spline et circle et autre
propose moi
